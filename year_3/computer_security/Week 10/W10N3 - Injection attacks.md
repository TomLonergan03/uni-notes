[[W10N1 - Cross-site scripting (XSS)|XSS]] and [[W10N2 - Cross-site request forgery (CSRF)|CSRF]] attacks are client side attacks, which focus on compromising a single user of a web application. Injection attacks instead target the web server, and attempt to gain unauthorised access to the sever itself.

An injection attack occurs when an application passes an input to an interpreter (e.g. SQL database request, system shell) without being properly sanitised. This allows an attacker to execute arbitrary code within the context of that interpreter by crafting specific inputs.
# OS injection
An OS injection attack involves executing commands on a shell where the command's arguments are not properly secured.
## Example
If we have a webserver that prints the result of the Linux program `whois`. It runs a PHP script on the server, and is invoked like this (with a form/script constructing the URL) `http://www.example.com/content.php?domain=google.com`, with an implementation of `content.php` which looks something like this:
```php
<?php
	if ($_GET['domain']) {
		<? echo system('whois '.$_GET['domain']); ?>
	}
?>
```
This could be attacked like this:
If we give the input `http://www.example.com/content.php?domain=google.com; rm -rf / --no-preserve-root`, this would be executed as `system(whois google.com; rm -rf / --no-preserve-root)`, as a `;` marks a separation between commands in bash. (As a side note, this command can literally [permanently brick a machine](https://thenextweb.com/news/running-a-single-delete-command-can-permanently-brick-laptops-from-inside-linux), in what is a bizarre quirk of how Linux treats UEFI bios data).

## Defences
1. **Sanitise inputs:** in the above example, we know that a URL cannot contain a semicolon, so the input should not be accepted
2. **Use the lowest privilege option:** we do not need to use `system()` to execute a `whois` command, as there are ways to execute a program without spawning a shell, e.g. `<? echo execv('whois '.$_GET['domain']); ?>`
3. **Escape inputs:** PHP (and most other programming languages) provide a method which converts any input into a string that is safe to use as a command argument by adding single quotes around a string and escaping any existing single quotes within the input. In PHP this function is `escapeshellarg()` and converts an input such as `www.google.com; rm -rf / --no-preserve-root` to `'www.google.com; rm -rf / --no-preserve-root'`, which would no longer be executed by bash.

# SQL injections
An SQL injection attack involves making unintended requests to databases used by a webserver, using injected SQL queries. This can modify records, or leak confidential information.
## SQL basics
SQL is used to query a database. An simple SQL query looks like `SELECT password FROM user_accounts WHERE username='alice'`, and when executed on this database:

| username | password |
| -------- | -------- |
| alice    | 01234    |
| bob      | 56789    |
| charlie  | 43210    |
would return `01234`.
A new record can be inserted using `INSERT INTO user_accounts VALUES ('eve', 98765)`, which would update the database to look like this:

| username | password |
| -------- | -------- |
| alice    | 01234    |
| bob      | 56789    |
| charlie  | 43210    |
| eve      | 98765    |
Other SQL commands include:
- `DELETE FROM table_name WHERE condition`: deletes existing records which satisfy the condition
- `DROP TABLE table`: deletes the entire specified table
- `--` or `#` marks the start of a comment (this is useful for injection attacks)
- `;` separate commands
## Example
A web server logs in a user if the user exists with a given username and password:
```php
$conn = pg_pconnect("dbname=user_accounts");
$result = pg_query(conn,
				  "SELECT * from user_accounts
				  WHERE username = '".$_GET['user']."'
				  AND password = '".$_GET['pwd']."';");
if (pg_query_num($result) > 0) {
	echo "Success";
	user_control_panel_redirect();
}
```
Here, an attacker could log in as admin by using the username `admin';--` and any password. This is as when the SQL query is assembled, it would look like:
```SQL
SELECT * from user_accounts
WHERE username = 'admin';
--'AND password = 'password';
```
which would return a record if an account with the username `admin` exists

They could also delete any specific user, with the username `admin'; DELETE FROM user_accounts WHERE username=alice;--`, or delete the entire table using the username `admin'; DROP TABLE user_accounts;--`.
## Defences
- **Input sanitation:** Most languages have SQL string escape functions also, in PHP it is `mysql_real_escape_string()`

