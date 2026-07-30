# Methodology

* **Run Responder & mitm6**: Start with Responder (run it all day) and mitm6 (carefully, as it can interfere with legitimate network traffic) for man-in-the-middle attacks to capture credentials and intercept traffic. Run them early in the morning before users log in to gather as much data as possible.
* **Run Scans**: Use scans to generate network traffic and identify active services.
* **Check Websites (http\_version)**: If scans are taking too long or yielding no results, focus on internal websites or web applications.
* **Default Credentials**: Test common default credentials on web logins (printers, Jenkins, etc.).
* **Think Outside the Box**: Look for other exploitable network services or misconfigurations when traffic or relays aren't available.
