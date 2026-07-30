# Reverse Engineering API Documentation

### Understanding API Endpoints

**API Endpoints** are specific URLs that handle requests for data or actions in an application. They serve as communication channels between different parts of an application, ensuring data flows correctly. For instance, endpoints like `/api/v1/user` may retrieve user profile data, while `/api/v1/comments` allows for posting or retrieving comments.

#### Importance for Penetration Testers

For penetration testers, API endpoints represent potential entry points for finding vulnerabilities. Each endpoint can be examined for security flaws, such as:

* **Authentication Issues**
* **Injection Flaws**
* **Data Leakage**

Identifying and analyzing these endpoints helps map out an application’s attack surface, uncovering weaknesses that could be exploited.

#### How to Identify API Endpoints

Identifying API endpoints is crucial for further analysis. Here are common methods:

1. **Manual Inspection**
   * Check the application’s documentation (e.g., `https://example.com/api/docs`).
   * Search for "API documentation" on the application’s website or Google.
2. **Traffic Interception**
   * **Manual Documentation using Postman**:
     * Create a Postman workspace to save collections.
     * Use the [Postman Interceptor extension](https://chromewebstore.google.com/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo?hl=en\&pli=1) to capture requests while interacting with the application.
   * **Automatic Documentation**:
     * Use [mitmproxy ](https://docs.mitmproxy.org/stable/overview-installation/)**(man-in-the-middle proxy)** to capture web traffic.
     * Save the captured requests and convert them into Open API 3.0 format using the tool [mitmproxy2swagger](https://github.com/alufers/mitmproxy2swagger).

#### Step-by-Step Process for Automatic Documentation

1. Run mitmproxy in the terminal to start capturing traffic.
2. Configure your proxy settings to match the port where `mitmweb` is listening (default is 8080).
3. Explore the target application to gather all traffic.
4. Save the captured requests from `mitmweb`.
5. Use the saved flow file to generate an Open API YAML file:

   ```bash
   sudo mitmproxy2swagger -i flows -o speca.yml -p http://example.api.com -f flow
   ```
6. Edit the YAML file as needed, ensuring to include any ignored endpoints.
7. Remove `ignore:` from endpoints you wish to include using a text editor.
8. Validate and correct formatting by running the `mitmproxy2swagger` script again, adding `--examples` flag for enriched documentation.

   ```bash
   sudo mitmproxy2swagger -i flows -o speca.yml -p http://example.api.com -f flow --examples
   ```
