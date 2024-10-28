java cWeek 13.10-15.10
Preconditions:
Previous assignment completed (instructions assume that you have a basic http server with a capability to receive and
send messages)
Tasks:
- Change your existing http server to https server
- Implement a basic user authentication
Background reading:
- https://en.wikipedia.org/wiki/HTTPS
- https://en.wikipedia.org/wiki/Basic_access_authentication
- https://en.wikipedia.org/wiki/List_of_HTTP_status_codes
- https://docs.oracle.com/en/java/javase/21/docs/api/jdk.httpserver/com/sun/net/httpserver/pack
age-summary.html
Classes needed in the implementation:
- com.sun.net.httpserver.HttpsServer
- com.sun.net.httpserver.HttpsConfigurator
- com.sun.net.httpserver.BasicAuthenticator
- com.sun.net.httpserver.HttpHandler
New tools to learn:
- https://docs.oracle.com/en/java/javase/21/docs//specs/man/keytool.html
Instructions
Step 1 - Create a self-signed certificate
You can create the certificate with following command:
keytool -genkey -alias alias -keyalg RSA -keystore keystore.jks -keysize 2048
Password should be strong but you need to remember as you need it later. The keytool will also ask your name, organization, location information etc. As your first and last name, you must give “localhost” without the quotation marks.
The command creates keystore.jks in to the folder you are currently in. Copy the keystore.jks in to your server project folder.
The keytool is part of the java jdk and if the command cannot be found, you need to either:
- add it to your PATH environment variable so you can use it easily just by writing “keytool”, or
- write the whole path before the “keytool” so it is then found and executed.
      
Step 2 – Configure server to use TLS/HTTPS
Remember to read the oracle documentation about http server listed in background reading section in first page.
After reading the documentation, simply copy the code just right after the sentence: “A simple example SSLContext could be created as follows:” and paste it to a new function called (such as, you can rename it if you want):
private static SSLContext myServerSSLContext() throws Exception After that, you need to:
- Import the necessary classes to the .java file (shown in errors since they are unknown to IDE before importing them).
- “passphrase” - replace this with the password you gave in Step 1 when you created your self signed certificate.
- “testkeys” - replace this with the key file name you used in the Step 1 — if you followed the instructions, it would be “keystore.jks”.
- As the function should return a SSLContext, do that. You already have the SSLContext object ssl in the code you copied. Return that to the caller
Note that putting password directly to the code is not generally a good idea. There are better ways to provide the passwords but for the coursework this is enough as we are testing our server locally.
Step 3 – Changing HTTP server to HTTPS server
Next, change the HttpServer to HttpsServer (see the difference!) and import that class. You can now remove the HttpServer import since it is no longer needed.
Next thing you need to do is to add code to main() to actually use the function you implemented above. Right after you created the HttpsServer, add this line:
SSLContext sslContext = myServerSSLContext();
So you now call the method you just implemented to create a SSLContext for your HTTPS server, using the self signed certificate you created in the Step 1.
Configure the HttpsServer to use the sslContext by adding this call to setHttpsConfigurator, which you give the sslContext you just created. Add this code right after the line above.
server.setHttpsConfigurator (new HttpsConfigurator(sslContext) { public void configure (HttpsParameters params) {
InetSocketAddress remote = params.getClientAddress(); SSLContext c = getSSLContext();
SSLParameters sslparams = c.getDefaultSSLParameters(); params.setSSLParameters(sslparams);

} });
Step 4 – Error handling
You need to add a try/catch structure to the main function to catch errors that might happen during code execution. You can always just throw the error and let the operating system sort the ending of the problem but that is bad for the person who uses and tries to understand why your program failed. Therefore, we use them to inform what went wrong and possibly log the information to a logger. For now, we just use console print.
If not yet there, add a try catch structure in the main, call all of the code you have there currently in the try block, and in the catch block, print out exception information, for example, just simply:
} catch (Exception e) { e.printStackTrace();
}
You might want to catch different exceptions in different catch blocks to do some more fine grained error handling, for example:
} catch (FileNotFoundException e) {
// Certificate file not found! System.out.println(“Certificate not found!”); e.printStackTrace();
} catch (Exception e) { e.printStackTrace();
}
More advanced logging and error handling will be discussed later during the course.
Step 5 – Test with curl
Note that by default, curl does not accept self signed certificates from a server. So you need to tell curl to accept them. Use an option -k (or longer --insecure). For more information, see https://curl.se/docs/sslcerts.html
Send a HTTPS POST request to the server, use:
UNIX:
curl -k -d 'Kauppatori' https://localhost:8001/info -H 'Content-Type: text/plain’
 
Or if you are using windows, you might need to replace the ‘ -> “
curl -k -d “Kauppatori” https://localhost:8001/info -H “Content-Type: text/plain”
And receive what you just sent to the server with command: curl -k https://localhost:8001/info
You may use the curl's -trace-ascii out.txt option to see how the client (curl) and the server communicate. Check the generated out.txt -file for details.
Step 6 – basic authentication
The second goal of this exercise is to add Basic HTTP authentication to the server. After this, users cannot POST or GET any messages unless authenticated.
Create new file called UserAuthenticator.java and add a new class to the server project, called UserAuthenticator. It must extend com.sun.net.httpserver.BasicAuthenticator.
We do not yet have a place to store user credentials. It will be implemented in a later exercise. For now, add a member variable Map to hold the usernames and passwords to the UserAuthenticator. Set it to null when you declare it. You could name the member variable as “users”:
private Map users = null;
Create a constructor method for the UserAuthenticator class. In the beginning, call super(), BasicAuthenticator’s constructor, giving “info” as the realm to apply the authentication to.
Also, in the constructor, instantiate the “users” map to be a Hashtable (Map is just an interface class, and Hashtable implements that interface). Then finally, add one dummy user in UserAuthenticator’s constructor to the “users” map:
users = new Hashtable();
users.put("dummy", "passwd");
Finally, implement the checkCredentials method inherited from the BasicAuthenticator class. There, you should check if, in the Hashmap, there is a username/password that matches the ones in the parameters of checkCredentials. If yes, return true, otherwise return false.
Now you have a simple basis for an authenticato代 写program、python/Java
代做程序编程语言r for the server. Later, users can register as users to the server, and the credentials they send are added to the Hashtable. Even later in the course, you will actually store this user information into a database, instead of using a Hashmap.
But where do you call these methods to verify the user? You do not call them directly. Instead, you tell to the context created earlier in the Exercise 1 to use this authenticator when serving client requests.
In the Server’s main method, just before creating the context, create an instance of the UserAuthenticator. You already have the code to create the context by calling server.createContext. Change this so that you take the return value of server.createContext function (we did not care about that earlier) to a HttpContext variable. Then set the authenticator for that context by calling setAuthenticator, giving your UserAuthenticator as a parameter.

Step 7 – Test the authentication
In curl you need to authenticate using the same username and password you created in the UserAuthenticator (assuming user “dummy” and password “passwd”, you may have something different there):
curl -u dummy:passwd -k https://localhost:8001/info
Note the -u (as in user) option and the username and the password given. The same applies obviously to POSTing messages, since the same basic authentication is applied to that realm too.
UNIX:
curl -u dummy:passwd -k -d 'Yliopisto' https://localhost:8001/info -H 'Content-Type: text/plain’ WINDOWS:
curl -u dummy:passwd -k -d “Nallikari” https://localhost:8001/info -H “Content-Type: text/plain”
Step 8 – Functionality for users to register
For registration, create a new HttpHandler with a new realm (or path) “/registration”. This is needed as info -realm expects the user to be authenticated and therefore it cannot accept new users (as they do not have the credentials yet).
Users can then register via this https: /localhost:8001/registration address, since the authentication implemented in Step 6 only applies to the “info” realm. So we are not going to set an authenticator for this new HttpHandler with “registration” realm. What you need to do is:
- Create a new class RegistrationHandler, which (like Server.java if you followed the course example) implements HttpHandler interface.
- Create a constructor for RegistrationHandler, that takes as a parameter a UserAuthenticator object.
- RegistrationHandler also needs to have a UserAuthenticator as a member variable so add it there.
- In the constructor of RegistrationHandler, assign UserAuthenticator to the member variable,
because the RegistrationHandler must be able to use the UserAuthenticator later when handling
POST requests from clients to authenticate users.
- Handle HTTP POST messages from clients, checking content type and content length etc. (see steps
below).
o ButdonothandleHTTPGETinRegistrationHandler!Wedonotwanttoprovideanyuser
information to the clients through this non-authenticated path — the client could be malicious, attempting to fish user information from registered users through this path that is not using authorization! You can respond to HTTP GET requests with error 400 and response body “Not supported”.
- For now, we assume that the clients send the registration information in the HTTP POST request body as a simple string “username:password”. (Later all content between client and the server will be JSON structures).
- Read the HTTP POST request body and parse the content. You can easily separate the two parts of “username:password” using Java String class’ split() method.
- Add the new user to UserAuthenticator from the two strings you got from calling String.split(). Obviously you want to check that the splitted string array has really got two strings, and if not, a

response to client is needed that data was not OK. You need to add a new method to the UserAuthenticator:
public boolean addUser(String userName, String password) { // TODO implement this by adding user to the Hashtable
}
Note: do not allow registering same username twice! This would actually lead to a situation where the existing user’s password is changed to the new one, since the data structure here is a Map. Map can only hold one key value, assigning a new element with a same key effectively just changes the value of the key- value pair -- the password! So check that the key does not exist and only then add the new user to the hash table.
Changing user credentials for an existing user should happen elsewhere (not in this handler) so that user is properly authenticated before allowing changing user data! Remember, users are not authenticated using HTTP Basic authentication when using the / registration URL path!
So if the username is already in the Hashtable, return false to the RegistrationHandler, which should pass the error to the client in the response! You can select which HTTP error code to pass to client, but for example 403, 405 might be good ones. Response body can further clarify to client what went wrong, for example: "User already registered”.
Finally, in the Server’s main function, instantiate the user registration Handler, when creating a new “/registration” Context
Step 9 – Test user registration with curl
You should be able to register new users with curl like this:
UNIX
curl -k -d 'dummyuser2:stillapassword' https://localhost:8001/registration -H 'ContentType: text/plain’ WINDOWS
curl -k -d “dummyuser2:stillapassword” https://localhost:8001/registration -H “ContentType: text/plain”
Note the path “/registration” in the address, and that content to the HTTP POST now has the username and password separated by “:”
Test also sending new locations with the created user.
Step 10 – sending the code to gitlab
After finishing your testing, make sure you replace the keystore location and password with args[0] and args[1] to allow the gitlab pipeline to replace both keystore location and password with server generated one. Your code will not compile if this has not been done (NOTE: in some cases it might actually work due

keystore being part of your commit and the password hardcoded in your code (especially if the keystore location and path is in project root folder), this might work but is unintended and can cause problems in future).
Additional tasks (refactoring code):
- Create a handler -class and move the handler implementation from Server class to it if you haven’t done so. It makes the project and code more clean. (For example HandleLocation -class, HandleRegistrations -class, etc...) Markus edit: unnecessary instructions, the original reference was to refactor the code IF the handle -class was implemented as part of the Server.java class where the main function was also present due examples.
Troubleshooting
keytool is not found.
- As explained in the instructions, find the JDK bin directory where it should be. Add that path to the computers PATH environment variable, or launch the tool with full path to the keytool binary.
When server launches, it cannot find the certificate file.
- If the file is without path, place it in the server root directory and launch the server from command line in that directory using command java -jar target/your-jarfile.jar.
- You can put the certificate file name with full path information, so it is found.
    
         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
