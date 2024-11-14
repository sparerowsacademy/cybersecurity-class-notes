**Understanding IDOR: A Quick Recap**

IDOR vulnerabilities occur when applications expose references to internal objects, such as database records, without sufficient access control checks. This flaw can allow attackers to access or modify data they shouldn’t be able to see or change. Common examples include altering URL parameters to access or edit records that belong to other users.

**Manual IDOR Discovery Process**

*Step 1: Identify Potentially Vulnerable Endpoints*

User Profile or Account Pages: Look at pages where users can view or edit their data (e.g., /user/profile?id=123).
Resource Access Links: URLs that use unique IDs to access resources like orders, messages, invoices, etc.
API Endpoints: Modern web apps often use APIs, which are commonly vulnerable if not secured. Look for API calls that use IDs to fetch or update resources (e.g., /api/order?id=456).

*Step 2: Test for Access Control Weaknesses*
Parameter Tampering:
Try modifying the id parameter in the URL or API request to another user's ID to see if you can access their data.
Example: Change GET /user/profile?id=123 to GET /user/profile?id=124 and see if it returns another user’s profile.
Session Hijacking:
Log in as User A, note down the resource URLs and IDs, then log out and log in as User B.
Try accessing User A’s resources (e.g., replace User B’s ID with User A’s ID in the URLs).
Review HTTP Methods:
Try different HTTP methods (GET, POST, PUT, DELETE) on endpoints to see if you can access or modify unauthorized data.

*Step 3: Look for IDOR in Common Application Functionalities*
Modify Profile Details: Try to update other users' profile details by changing the user_id parameter.
View Transactions/Orders: Try to view other users’ transaction details by changing the order or transaction ID.
Delete Operations: If there is a delete feature, attempt to delete another user's data by changing the ID in the request.

**Automating IDOR Discovery**

Automation can help streamline the process of testing many endpoints quickly. Here’s a step-by-step approach to automating IDOR detection.

*1. Setting Up Burp Suite for IDOR Automation*

Burp Suite’s Intruder and Repeater tools can help automate the process of testing IDORs.

Burp Intruder:
Set up the target request with the parameter you want to test (e.g., id=123).
Use Intruder to send multiple requests, modifying the id parameter in increments (e.g., 123, 124, 125, etc.).
Check the responses for any differences that might indicate access to other users’ data.
Burp Extensions:
Autorize: This Burp extension automates authorization testing by comparing responses with different sessions. After logging in as User A, it will compare responses when the same request is made with User B’s session.
Turbo Intruder: If you need high-speed fuzzing, Turbo Intruder can help in testing many IDs rapidly.

*2. Using Python for IDOR Automation*

You can also create a Python script to automate IDOR testing. Here’s a simple example:

python
Copy code
import requests

# Define the target URL and session cookies
url = "https://targetsite.com/user/profile?id="
cookies = {
    'session': 'your-session-cookie'
}

# Define the range of IDs to test
for user_id in range(123, 130):  # Adjust range as needed
    response = requests.get(url + str(user_id), cookies=cookies)
    if "unauthorized" not in response.text.lower():  # Check for access
        print(f"IDOR Found! Access to ID {user_id}")
        print(response.text)
    else:
        print(f"Access denied for ID {user_id}")
This script sends requests to https://targetsite.com/user/profile?id=, iterating through different id values and printing the response if it detects unauthorized access.

*3. Using FFUF (Fuzz Faster U Fool) for ID Enumeration*

FFUF is a powerful tool that can help with brute-forcing parameters and discovering IDOR vulnerabilities.

Install FFUF:

bash
Copy code
sudo apt install ffuf
Run FFUF to Fuzz IDs:

bash
Copy code
ffuf -u https://targetsite.com/user/profile?id=FUZZ -w ids.txt -c -mc 200
-u: Specifies the URL to test, where FUZZ is replaced by each entry in the wordlist.
-w: Specifies the wordlist. You can create a list of user IDs you want to test, e.g., 123, 124, 125.
-mc: Match status code 200, which indicates a successful request.
4. Using Tools like Autorize for Session Comparison
To test if a user can access unauthorized data, you can use the Autorize Burp extension, which automates this by comparing responses for different users.

Setup: Open Autorize in Burp Suite and set your User A session token in Autorize’s configuration.
Run Requests: While logged in as User B, make requests to potentially vulnerable endpoints.
Analyze Results: Autorize will highlight any instances where User B’s session could access User A’s data.
5. Further Automation with Tools Like Arjun
Arjun can help automate parameter discovery, which can reveal hidden parameters that may be vulnerable to IDOR.

Install Arjun:

bash
Copy code
pip install arjun
Run Arjun:

bash
Copy code
arjun -u https://targetsite.com/profile
Arjun will test for various parameters and may reveal undocumented or hidden parameters that could be vulnerable to IDOR.

**Best Practices and Tips for Students**

Session Management: Remind students to check for proper session handling, as improper session management often leads to IDOR.
Use Rate Limits: When automating, remind students to respect rate limits to avoid detection or being blocked.
Report Responsibly: If students find any real IDOR vulnerabilities, remind them to report it responsibly as part of ethical hacking.

Automated parameter discovery is essential when you’re hunting for vulnerabilities like IDOR, as it helps reveal hidden or undocumented parameters in web applications that may expose sensitive data or allow unauthorized actions. Tools like Arjun and ParamMiner in Burp Suite make this process easier. Here’s a detailed overview of how these tools work and how to use them effectively.

**Why Parameter Discovery Is Important**

Web applications often have hidden parameters that aren’t visible in the front-end interface or documented in the API documentation. These parameters can control access to resources, modify data, or perform sensitive actions. Discovering these hidden parameters can reveal potential vulnerabilities, such as IDOR, SSRF, access control issues, and more.

*Tool 1: Arjun*

Arjun is a command-line tool designed for finding hidden HTTP parameters in GET and POST requests. It’s highly effective because it sends hundreds or thousands of requests with different parameter names to check if the server responds to any of them.

Step-by-Step Guide to Using Arjun
1. Install Arjun
If you haven’t already installed Arjun, you can do so using pip:

bash
Copy code
pip install arjun
2. Running Arjun for GET Requests
Suppose you have a URL like https://example.com/profile, and you want to discover parameters that might reveal user-specific information.

bash
Copy code
arjun -u https://example.com/profile
Explanation: By default, Arjun tests common parameters (like id, user, profile_id) on the specified URL. It sends requests with each parameter, and checks for responses that may indicate the presence of a parameter, such as non-404 status codes or significant changes in response size.
3. Running Arjun for POST Requests
Arjun can also be used to find parameters in POST requests, which are common in forms and APIs.

bash
Copy code
arjun -u https://example.com/api/update_profile -m POST
Explanation: Here, -m POST specifies that we’re sending a POST request to test for parameters. Arjun will use a wordlist of common parameters to see if the server responds differently, revealing hidden fields that may be accessible.
4. Advanced Options with Arjun
Output to a File: You can save the results to a file for later analysis:

bash
Copy code
arjun -u https://example.com/profile -oJ output.json
This will output the results in JSON format for easier parsing.

Custom Wordlist: Arjun has its own list of common parameters, but you can use a custom wordlist if you want:

bash
Copy code
arjun -u https://example.com/profile -w custom_params.txt

*Tool 2: Burp Suite’s ParamMiner*

ParamMiner is a Burp Suite extension that automates the discovery of hidden or additional parameters in web requests. It can reveal undocumented parameters that the server may accept but are not shown in the HTML or JavaScript code.

Step-by-Step Guide to Using ParamMiner
1. Install ParamMiner in Burp Suite
Go to the Extender tab in Burp Suite.
Go to the BApp Store and search for ParamMiner.
Install ParamMiner by clicking the install button.
2. Configure ParamMiner
Once installed, you can configure ParamMiner:

Automatic Mining: Enable automatic mining on all requests. ParamMiner will automatically check each request you send through Burp for hidden parameters.
Context Menu Mining: Right-click any request in the Proxy or Repeater tabs and select Guess Headers or Guess Parameters from the context menu to start mining for that specific request.
3. Running ParamMiner on Specific Requests
Send a Request to Repeater: Identify a request you want to test, such as a request to access or update profile information.
Right-click the request and select Guess Parameters.
Analyze Results: ParamMiner will send requests to test common parameter names (like id, user, account_id). If the server accepts any of these parameters, you’ll see changes in the response.
4. Guessing Headers with ParamMiner
Some hidden functionalities may be triggered by specific HTTP headers (like X-User-ID, X-Original-URL). You can also use ParamMiner to discover these.

Right-click a request and select Guess Headers.
ParamMiner will try various header names and look for changes in the server’s response.

*Tool 3: FFUF (Fuzz Faster U Fool)*
While primarily a web fuzzer, FFUF can be used for parameter discovery as well, by fuzzing GET and POST requests with different parameter names.

Step-by-Step Guide to Using FFUF for Parameter Discovery
1. Install FFUF
If you don’t have FFUF installed:

bash
Copy code
sudo apt install ffuf
2. Create a Wordlist for Parameters
FFUF requires a wordlist. You can use pre-made wordlists, like those in SecLists, or create a custom one for parameters you suspect might be used.

3. Run FFUF for Parameter Discovery
Assuming you’re testing the endpoint https://example.com/profile for hidden parameters:

bash
Copy code
ffuf -u https://example.com/profile?FUZZ=1 -w /path/to/parameter_wordlist.txt -c
FUZZ is a placeholder that FFUF replaces with each word from your wordlist.
The response size or HTTP status code can indicate if a parameter is accepted by the server.

*4. Testing POST Requests with FFUF*

You can also use FFUF to discover POST parameters by specifying -X POST and including a data payload:

bash
Copy code
ffuf -u https://example.com/api/update_profile -X POST -d "FUZZ=1" -w /path/to/parameter_wordlist.txt -c
Tool 4: Kiterunner (for API Parameter Discovery)
Kiterunner is designed for API fuzzing and parameter discovery in web services that use a RESTful or GraphQL API. It allows you to search for undocumented parameters and endpoints.

Step-by-Step Guide to Using Kiterunner
1. Install Kiterunner
You can download it from GitHub:

bash
Copy code
git clone https://github.com/assetnote/kiterunner.git
cd kiterunner
make
2. Run Kiterunner Against an API
If you have an API endpoint, say https://example.com/api, you can use Kiterunner with a wordlist:

bash
Copy code
kr scan https://example.com/api -w /path/to/api_wordlist.txt
Kiterunner will test each word from the wordlist to discover any hidden or undocumented parameters in the API.

Tips for Effective Parameter Discovery
Use Multiple Wordlists: Try combining different wordlists (like parameter names, common API parameters) to improve coverage.
Check Response Differences: Look for variations in response codes, sizes, and content. A change in these might indicate a discovered parameter.
Test Both GET and POST Methods: Some parameters might work only in one method, so always test both.
Combine Tools: Using tools like Arjun for quick scans, ParamMiner for in-depth analysis, and FFUF or Kiterunner for more extensive fuzzing can yield comprehensive results.
