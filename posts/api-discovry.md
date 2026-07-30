# API Discovry

### **Web API Indicators**

#### **Common API URL Patterns**

* **URLs:**
  * `https://target-name.com/api/v1`
  * `https://api.target-name.com/v1`
  * `https://target-name.com/docs`
  * `https://dev.target-name.com/rest`

#### **Directory Names to Look For**

* `/api`
* `/api/v1`
* `/v1`, `/v2`, `/v3`
* `/rest`
* `/swagger`, `/swagger.json`
* `/doc`, `/docs`
* `/graphql`, `/graphiql`, `/altair`, `/playground`

#### **Subdomains Indicating API Use**

* `api.target-name.com`
* `uat.target-name.com`
* `dev.target-name.com`
* `developer.target-name.com`
* `test.target-name.com`

#### **HTTP Response Indicators**

* Look for messages such as:
  * `{"message": "Missing Authorization token"}`

#### **Third-Party Sources for API Information**

* **GitHub**: Search for API documentation or implementations.
* **Postman Explore**: Explore public APIs and their documentation.
* **ProgrammableWeb**: Find categorized APIs.
* **APIs Guru**: A curated list of APIs.
* **Public APIs GitHub**: A collective repository of free APIs.
* **RapidAPI Hub**: Access to thousands of APIs.

#### **Passive Reconnaissance**

Passive reconnaissance involves gathering information without direct interaction, typically relying on Open Source Intelligence (OSINT).

**Tools/Sites for Passive Recon**

* **Google Dorking**: Use advanced Google search techniques to discover APIs.\
  **Example queries**:
  * General: `target API`, `target API docs`
  * Specific: `inurl:"/api/v1" site:target.com`
  * Technology-focused: `intitle:json site:target.com`
  * Additional queries:
* **GitDorking**: Search GitHub for API-related files and information.\
  **Useful search terms**:

  * `filename:swagger.json`
  * `extension:.json`
  * Keywords: `"api key"`, `"authorization: Bearer"`, `"access_token"`, `"secret"`, `"token"`

  **GitHub Tabs to Check**:

  * **Code Tab**: Look for relevant files and keywords in the code.
  * **Issues Tab**: Check for unresolved issues that may involve exposed keys.
  * **Pull Requests Tab**: Review proposed changes for potential exposed APIs.
* **Shodan**: Utilize Shodan to find open APIs and gather details about open ports.\
  **Example queries**:
* **Wayback Machine**: Access archived web pages to find old or deprecated API endpoints (Zombie APIs).
* **TruffleHog**: Automate the discovery of exposed secrets in GitHub repositories.\
  **Usage Example**:

  ```bash
  sudo docker run -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --org=target-name
  ```

#### **Active Reconnaissance**

Active reconnaissance involves directly interacting with your target, often through scanning to uncover APIs and gather actionable information.

**Tools/Sites for Active Recon**

* **Nmap**: Identify open ports and enumerate HTTP services.

  ```bash
  nmap -sV --script=http-enum <target> -p 80,443,8000,8080
  ```
* **Amass**: Discover active subdomains and filter for API endpoints.\
  *(Don’t forget to include your API keys to check available services using `amass enum -list` command.)*

  ```bash
  amass enum -active -d target-name.com | grep api
  ```
* **Gobuster**: Use Gobuster with an API-specific wordlist to find directories on a target.

  ```bash
  gobuster dir -u target-name.com:8000 -w /home/hapihacker/api/wordlists/common_apis_160
  ```
* **Kiterunner**: Discover API endpoints using various HTTP methods.\
  **Quick Scan**:

  ```bash
  kr scan http://target.com -w ~/api/wordlists/data/kiterunner/routes-large.kite
  ```

  **Replay Requests**:

  ```bash
  kr kb replay "GET .../api/privatisations/count" -w ~/api/wordlists/data/kiterunner/routes-
  ```
