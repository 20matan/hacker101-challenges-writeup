# hacker101-challenges-writeup
in this repository i write the solutions for the hacker101 challenges. 
If you have some better solutions, feel free to share!

**Note: for those challenges you should use web browser without XSS auditor. I used firefox.** <br>
**I might miss some exploits, so feel free to suggest some**

### Level 0: Breakerbank
1. _**CSRF**_:
Viewing the page source, there is no CSRF protection. so we can create POC of CSRF for sending money to our account.<br>
you use burp/any other tool for that.
2. _**IDOR**_:
Looking at the transfer money request, we see there are "to" and "amount" fields. if we intercept the request, and change the "to" to our bank id, and add `from=1` in the body payload, we can transfer money from the bank account with the id 1 to our account.
3. _**Reflected XSS**_:
Clicking on the `personal payment link`, we navigate to a site with url param `to`, and the value of this to param is inside the `Destination account` input. But, this field is not vulnerable to XSS (or at least i didnt succeed to).<br>
But, adding to the url `...&amount=value`, we see that this value is in the amount field. so using the payload `...&amount="><img src="/" onerror="alert(1)"`, will result in alert popup.
4. _**Reflected XSS**_:
Using the same payload we used in 3, we place this payload in amount input field and press `Transfer`. it will result in validation error about not being integer, and an alert popup.<br>
(They wrote in the source that they have 4 vulns. im not sure if this is the 4th, but i couldnt find anything else. so if you found something else, lmk)


### Level 1: Breakbook
1. _**CSRF**_:
Althought there is a csrf token in the post comment, the backend only validates the token length (32). so passing any string with the same length, will result in succesfully csrf vulnerability.
2. _**Stored XSS**_:
Following the first comment we see, it looks like every link in the comment becomse `<a>`.<br>
So using the payload `http://a.b.c"/onmouseover="alert(1)`, when user will hover his mouse over our comment, the script will be executed.
3. _**Forced Browsing**_:
Clicking the `Permalink` for any comment, will navigate us to `/post?id=1337` (real story with the 1337<br>
Changing the id to any other id, will allow us to see what posts other users wrote.<br>
Moreover, passing an id that is not exist in the system, will allow us to see the Trceback.
4. Im not sure what is the fourth vuln (they wrote there are 4). I didnt succeed to find another XSS vuln, so my guess is the 4th is csrf for post details.

### Level 2: Breaker Profile
1. _**Stored XSS**_:
We can run code through the description of the user. if we change it to something like `... [green"onmouseover="alert(1)" | text1 ] ...`, mouse hovering the text1 will result in alert popup.
2. _**Stored XSS**_:
We can run code using the Profile picutre URL. notice that it validate that the URL ends in PNG/JPG/ICO, so using the payload `"><img src="/" onerror="alert(1)" a=".png` will result in alert popup when we look at our profile.
3. _**Stored XSS**_:
The same as 2, but when we are in the edit page.
4. _**Reflected XSS**_:
Pressing the `Link to your profile` link will navigate us to a page with the url `...?id=OUR_ID`. changing this id to `blablabla`, will lead us to page which reflects our blablabla id. so using the url `?id=a<script>alert(1)</script>` will lead to XSS.
5. _**Reflected XSS**_:
In the edit profile, there is `id` url param. this id is reflected to the `action` attribute in the form tag.<br>
changing the URL to `...?id=1"><img src="/" onerror="alert(1)`, will result in the alert popup.
6. _**Unrelated Bonus**_:
Ok, so they wrote there is this "Unrelated Bonus", so im not sure this is the vuln, but i couldnt find anything better: <br>
We can place any domain in the image url, which allows us to serve an image with malicious payload that contain js.

### Level 3: Breaker CMS
1. _**Improper Authorization**_:
Looking at the source code, it seems that the code will show `<a>` if the user has a cookie with `admin=1` value in it.<br>
So after changing the admin=0 to admin=1 in the cookie, a new link appear.
2. _**Stored XSS**_: 
After clicking the `Edit page` link, we can edit the page title and body.<br>
placing in both the payload `<h1>a</h1>`, shows that the body is vulnerable to XSS, but it blocks the `<script>` tag.<br>
so using the payload `<img src="/" onerror="alert(1)">`, we succesfully see the alert popup.
3. _**Stored XSS**_:
Looking at the page source, we can see that the `<title>` tag contains the page title (the one we can edit).<br>
so changing the title to `</title><script>alert(1)</script>`, will result in succesfull XSS.
4. _**CSRF**_:
The edit page form has no CSRF defense mechanism.

### Level 4: Breaker News
Sorry, but i didnt do this level. when i entered the challenge, i got like 10 alerts. seems like there is no sandbox for each user, so i prefered to move on to the next challenges.

### Level 5: Document Repository
1. _**Reflected XSS**_:
Clicking on the `ebooks/` link, we navigate to `URL?path=/ebooks`, and we have a title in the document `Document Repository -- /ebooks/`. this looks like an options for reflected XSS, and indeed it is.<br>
changing the path url param to `<script>alert(1)</script>`, we will see the alert popup.
2. _**Path Traversal**_:
Seems like the site lists the files in the directory we supply through "path" url param.<br>
Changing the path to `../../../../../../../../etc`, will list all the files in this dirctory.<br>
**tip: when you are in the root, you can keep do more '../'s. if you are already in the root dir, you will still be in the root dir. so when you have path traversal, just do some more ../ to be more sure you reach the root dir**
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

### Level 8: Document Exchange
1. _**Reflected XSS**_:
The mime type value is vulnerable to reflected XSS. intercepting the request and changing the Content-Type from `image/jpeg` (or whatever file type you try to upload) to `Content-Type: image/jpeg<img/src="a/"/onerror="console.log(1)">` will result in XSS
