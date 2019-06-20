#	Cross-Site Request Forgery 

-	CSRF is an attack
-	CSRF forces authenticated users to run (execute) unwanted actions on the web application
-	CSRF attacks specifically target state-changing requests 
-	Not theft of data, since the attacker has no way to see the response to the forged request
-	With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker's choosing
-	If the victim is a normal user, a successful CSRF attack can force the user to perform state changing requests like transferring funds, changing their email address, and so forth
-	If the victim is an administrative account, CSRF can compromise the entire web application