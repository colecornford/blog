---
title: "Code Quality vs Code Security" 
date: 2020-01-07T00:29:41-06:00 
---

Today, I was reviewing some source code for security weaknesses. I stumbled across code that made me chuckle. Something along these lines.

```java
public String XOR(String method, String input, String key)
{
    if (method == "Encrypt") {  
        return XOR_Encrypt(input, key);
    }
    else if (method == "Decrypt") {
        return XOR_Decrypt(input, key);
    } else {
        return input;
    }
}
```

As software engineers, we have all contributed in our own way to the _legacy_. I remember during my university days I wrote an accounting assignment program in C#. I don't have the code anymore, but i clearly remember what the problem was. Have a look below.

```c#
public static void addBalanceCole(int amount){
    // adds balance to Cole
}

public static void addBalanceNina(int amount){
    // adds balance to Nina
}

class Person {
    public static int balance{ get; set;}
    public static string firstName{ get; set;}
    public static bool good{ get; set;}
}
```

While there are several issues with the above code, my specific issue surrounded the *static* keyword. You see, I kept getting an error:

`Static methods cannot access non-static members of Person.`

I dealt with this by making everything static, it made sense to me at the time. I scored **3/10** because Cole and Nina apparently shared the same balance, name, and whatever _good_ meant too. Obviously, not the best outcome.

So, what am I getting at? See, as engineers, we take pride in our work. When low-quality code gets pushed into production.. it stings. Not only because of our pride, but also because we know of the repercussions. Either you're called up Monday 2am for an incident, or someone 3 years down the road will. Great way to get some `git blame` kudos. 

So, to help stop the introduction of these bugs, the software industry built tools and processes to assist their developers.

There are plenty of articles that discuss in detail how to become a 10x developer. But today I wanted to talk about AppSec fixes in that context instead.

# The problem

Traditional enterprise security testing tends to occur towards the end of a project lifecycle. While you get do meet the occasional _Agile PM_ and interact with a _DevOps team_ **(?)**, the core operating model of older enterprises in Australia largely resembles Waterfall. 

![Beautiful Waterfall](../../img/posts/2020-01/waterfall.jpg)

This leads to a situation where infosec may block projects or at least the perception may exist. Business makes the call to either mitigate or accept the risk. In either situation, the outcome is middling...

* Risk Mitigation: Developers are incentivised to implement quick fixes as deadlines approach. Either they follow the advice of security consultants who lack business context or a comprehensive understanding of the technology stack. That, or developers try to identify the most time-effective fixes they can, usually on Stack Overflow or the OWASP CheatSheets.

* Risk Acceptance: The project continues unabated. However, accepting a risk does not fix the vulnerability. It's a decision the business has made at this point in time, but as we all know, it's only a matter of time...

The traditional code quality controls (Linters, Peer Review, Refactoring, etc) become weakly enforced for security fixes, and this impacts the future maintainability, reliability, and performance of an application. 

**Because of this, I wanted to highlight why it's important to meet or exceed the standard, even when under time pressure.**

## New Weaknesses

Sometimes, a security fix may introduce new vulnerabilities.

In this example, we have a Path Manipulation fix for a filename. A whitelist has been introduced. It reasonably defines the accepted filename format and prevents many potential attacks. 

Unfortunately, the Regex is vulnerable to Catastrophic Regex Backtracking.  If an attacker crafted a long input string that matched the Regex, it would cause a denial of service to this application due to the way that regex performs validation. Whether this is better or worse for the application will depend on the business and operating context, but regardless an unintentional flaw was introduced by a fix.

```java
import java.util.regex.*;  
public static boolean isValidFilename(String filename) {
    String filenamePattern = "^([A-Za-z0-9]+)*[.]jpg";
    Pattern r = Pattern.compile(pattern);
    Matcher m = r.matcher(filename);
    if (m.find()) {
        return true;
    }
    return false;
}
```

Another example of introducing a flaw via a fix comes from the eBay attack. As quoted from [Mitre's CWE Page for _Overly Restrictive Account Lockout Mechanism_](https://cwe.mitre.org/data/definitions/645.html).

> A famous example of this type of weakness being exploited is the eBay attack. eBay always displays the user id of the highest bidder. In the final minutes of the auction, one of the bidders could try to log in as the highest bidder three times. After three incorrect log in attempts, eBay password throttling would kick in and lock out the highest bidder's account for some time. An attacker could then make their own bid and their victim would not have a chance to place the counter bid because they would be locked out. Thus an attacker could win the auction.

In this situation, a control was implemented to address one kind of security vulnerability, but left the application exposed to an entirely different one. Without context of how the application is being used and what's important to the business functioning, it's difficult to introduce controls without some kind of oversight being made.

If you review the code below, you can see exactly how simple it is to quickly introduce a control without realising the implications of doing so.

```java
public static boolean login(String filename) {

    String uname = req.getParameter("username");
    if (crypto.bcrypt(req.getParameter("password")) == getHashedPass(uname)) {
        if (!isDisabled(uname)){
            login(uname);
            rmFailedLogins(uname);
        }
    }
    else {
        incrementLoginFails(uname);
        if (getLoginFails(uname)) > 3) {
            disableAccount(uname);
        }
    }
}
```

## Performance Issues

Performance is critical for some businesses. Keeping the user on your website. Running a scientific simulation. Being able to respond to emergencies. Performance matters across all industries. If your application starts to stutter when security controls are implemented, then investigate and contact your business owner about priorities.

Typically, performance issues arise from one of the following:
* Compute
* Memory
* Network
* Storage
* Humans

If you introduce a patch for speculative execution on your hardware, you may cause Compute impacts. If you put devices that inspect traffic, such as a WAF or an IPS, then network performance might degrade. How you manage encryption and logging can impact your storage read/write and memory usage. Is multi-factor authentication slowing your employees down? These are all relative to the context of the application and how your business operates using it.

I have some strong examples in the Cryptography section below that can help illustrate performance trade-offs better.

## Weak Control

Sometimes ineffective controls are implemented. This is common when developers follow stack overflow advice as gospel. Take this code for example. We have a regular expression being used to validate a description field. Unfortunately, description fields are generally freeform text and our users would complain if we banned special characters as they're part of written english. So we've put an arbitrary length limit on the field and left it at that. This from a glance appears to mitigate the risk as validation is being performed, in line with other fields. But even basic attack strings are being allowed through a weak control like this. 

In this case, there are more options that should have been investigated. Whether it was relying on a templating engine, using output encoding, or defining a strong content security policy, the current control is ineffective.

```java
import java.util.regex.*;  
public static boolean isValidDescription(String filename) {
    String filenamePattern = "^.+{0,512}"
    Pattern r = Pattern.compile(pattern);
    Matcher m = r.matcher(filename)
    if (m.find()) {
        return true
    }
    return false
}
```

Speaking of CSP, their implementation matters. Having a CSP does not always provide security. Here's an example from Ars Technica, a technology news website. This CSP is preventing HTTP domains from being used but otherwise achieving nothing. With unsafe-inline and unsafe-eval turned on, we have a misconfigured and ultimately, useless security feature as far as XSS is concerned.

```js
content-security-policy: default-src https: data: 'unsafe-inline' 'unsafe-eval'; 
child-src https: data: blob:; 
connect-src https: data: blob:; 
font-src https: data:; 
img-src https: data:; 
media-src blob: https:; 
object-src https:; 
script-src https: data: blob: 'unsafe-inline' 'unsafe-eval'; 
style-src https: 'unsafe-inline'; 
block-all-mixed-content; 
upgrade-insecure-requests
```

## Maintainence Nightmare

When under pressure to deliver, corners are cut and code quality suffers. This can mostly be seen in people abandoning time-tested practices like KISS, DRY, and YAGNI. In this example, we have a bunch of situations where a Null Dereference can occur. 
```java
class dbClass() {
    public static void addToDB(String DB_URL, String USER, String PASS, String QUERY)
    {
        Connection conn = null;
        Statement stmt = null;
        try{
            conn = DriverManager.getConnection(DB_URL,USER,PASS);
            if (conn == null) {
                System.out.println("Connection is Null " + conn.toString());
                return;
            }
            stmt = conn.createStatement(QUERY);
        }catch(Exception e){
            e.printStackTrace();
        }finally{
            stmt.close();
            conn.close();
        }
    }
}
```
To address this, the developer is checking for null before each object is accessed. While this approach will remove a bunch of the security findings from those annoying SAST tools, the code becomes a mess. Look at it. When code ends up like this, it's the first step to a maintenance nightmare. Your best bet is to look at front-loading the edge cases, and running your fixes through linting software.

```java
class dbClass() {
    public static void addToDB(String DB_URL, String USER, String PASS, String QUERY)
    {
        Connection conn = null;
        Statement stmt = null;
        try{
            if (DriverManager == null) {
                throw NullPointerException("DriverManager is null");
                return;
            }
            conn = DriverManager.getConnection(DB_URL,USER,PASS);
            if (conn == null) {
                throw NullPointerException("conn is null");
                return;
            } else {
                stmt = conn.createStatement(QUERY);
            }
        }catch(Exception e){
            if (e == null){
                throw NullPointerException("e is null");
                return;
            }else{
                e.printStackTrace();
            }
        }finally{
            if (stmt == null){
                throw NullPointerException("stmt is null");
                return;
            }else{
                stmt.close();
            }
            if (conn == null){
                throw NullPointerException("conn is null");
                return;
            }
            conn.close();
        }
    }
}
```
This is in a similar book to above, but a little different. It's the first google search for "Email Regex". While impressive looking, this string is ultimately a problem. Troubleshooting edge cases, altering the pattern, or just trying to interpret it is immediately difficult.

```java
public class emailValidator(String email) {
    String emailPattern = "(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*|"(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21\x23-\x5b\x5d-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])*")@(?:(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?|\[(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?|[a-z0-9-]*[a-z0-9]:(?:[\x01-\x08\x0b\x0c\x0e-\x1f\x21-\x5a\x53-\x7f]|\\[\x01-\x09\x0b\x0c\x0e-\x7f])+)\])"
    Pattern r = Pattern.compile(emailPattern);
    Matcher m = r.matcher(email)
}
```

## ... and Cryptography

Cryptography is considered such a specialist topic that even security consultants lack knowledge deeper than "use AES/RSA and HTTPS is good". Without a more detailed understanding of the implications for algorithm selection, you could significantly impact all aspects of your application. Some examples of things that could go wrong with a surface-level understanding of cryptographic algorithms:

* Typically, hashes are recommended for password fields in databases. As passwords are highly sensitive pieces of information, we want hashing algorithms that are resistant to being computed quickly in case the database is compromised. But hashes are also used for file integrity, to make sure nothing has been tampered with. Weak hashing algorithms are used as they are performant, security is not relevant for this context, and collissions are unlikely to matter. Using a strong hashing algorithm introduces no security improvement and a significant performance overhead.

* What mix of crypto methods you use matters. If your application transmits a tonne of tiny requests (such as a high frequency trading app, or an online multiplayer game), then the overhead of grabbing an RSA key each packet will have major consequences to the performance of your business. Similarly, managing how you do key-sharing for symmetric cryptography matters as well. You can take the SSL Handshake approach, which adds even more of a performance bottleneck initially, but might result in better performance as the symmetric key is shared between client and server.

* Whether you even need encryption is important too. Take for example a legal application. If the app encrypts the data before sending it to you, then it'll be doubly protected over the wire, and protected at rest too. This could be relevant in the case of physical device compromises. But consider if you did the same for a company such as Netflix or Youtube. Whenever data is encrypted it typically becomes uniformly distributed. These companies thrive on sending high quality video to users with limited to no-buffering. If the data was encrypted, then you wouldn't be able to take full advantage of compression, delivering a significantly worse product to your consumers.

## In Conclusion

This has been a bit of a longwinded post and thank you for sticking to the end. As a security engineer, you need to be aware of the impact that your controls will have on the lifecycle of an application. From cradle to grave, any changes you make will have material impact to the application and all current and future developers working on it. Keep your chin up, and keep learning. :)