# hacker101-challenges-writeup
in this repository i write the solutions for the hacker101 challenges. 
If you have some better solutions, feel free to share!

**Note: for those challenges you should use web browser without XSS auditor. I used firefox.** <br>
**I might miss some exploits, so feel free to suggest some**


### Level 5: Document Repository
1. _**Reflected XSS**_:
Clicking on the `ebooks/` link, we navigate to `URL?path=/ebooks`, and we have a title in the document `Document Repository -- /ebooks/`. this looks like an options for reflected XSS, and indeed it is.<br>
changing the path url param to `<script>alert(1)</script>`, we will see the alert popup.
2. _**Path Traversal**_:
Seems like the site lists the files in the directory we supply through "path" url param.<br>
Changing the path to `../../../../../../../../etc`, will list all the files in this dirctory.<br>
**tip: when you are in the root, you can keep do more '../'s, how much you want, you will still be in the root directory. so when you have path traversal, just do some more ../ to be more sure you reach the root dir**
3. _**Command injection**_:
It looks like `Search in directory` form search inside all the files in the current dir for the string we pass. so i looked at google for `linux search string in files`, and seems like the command it run on the server is something like `grep -rnw 'DIRECTORY' -e 'TEXT_TO_SAERRCH'`.<br>
Moreover, seems like if no matches were found, it returns empty page. so to exploit this feature, i used the payload `q" && ls -la"`, and on the bottom of the list i indeed saw a result of `ls -la` command.

### Level 6: Student Center
1. _**Reflected XSS**_:
Using the filter input, we see the value we place there will be shown to the url in the next screen. so using malicious payload, like `a"><img src="/" onerror="alert(1)"`, will resolve in a message alert
2. _**Stored XSS**_:
Let's try to check the firstname & lastname fields of the students.<br>
In edit page, those values are placed in input tags. if we place malicious payload `"><img src="/" onerror="alert(1)">` in the last name and submit, the next time we will edit this same student, we will have an alert popup.
3. _**SQL Injection**_:
The filter form is vulnerabile to SQLi, using the payloads `qq' OR 1='1--` will result in returning all the students.
4. _**CSRF**_:
The filter form has no CSRF token, you can easily generate CSRF page (using burp/any other tool) and see it.
5. _**IDOR**_ (Not listed in the vuln list):
When we go to the edit student list, the student id is passed through the url param.<br>
Changing the id to an id that is not listed in our students table, like `1111`, will result in a student data, probably there are students for every one who try to solve this challenge. we will not be able to edit the student details, but still we can see it.

### Level 7: Guardian
1. _**Reflected XSS**_: 
So the first thing we will try is XSS.<br>
Password field was vulnerable to XSS; using the payload `"><img src="/" onerror="alert(1)`, i've succesfully saw the alert popup.
2. _**SQL Injection**_:
We will use the default admin value that we had, and using random password, we will get the informative 'Invalid password', and using username other than `admin`, we get `User does not exist`.<br>
using the payload `QQ' OR 1='1--`, we get the message 'Invalid password', which means we bypassed the username filter in the sql query.<br>
using the payload `'` as the username, we can see the sql query;
It tries to get the password of the user with the username we pass. so, using the payload `=QQ' UNION SELECT '12345`, we return to the query result the password '12345', and if we send this password in the form, we bypass the authentication
