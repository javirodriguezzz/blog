---
title: "Search automation in Shodan"
date: 2022-03-24T20:03:28+02:00
draft: false
---
{{< justify-text >}}
# ¿What is *Shodan*? :male_detective:

[Shodan](https://www.shodan.io/) is a search engine which uses a database of billions of publicly available IP addresses from a large number of systems, specially IoT devices. It can find anything that connects directly to te internet and if its devices aren't protected, so can tell everything people with bad intentions need to know to break into your network. 

In this case, a Python script was developed to perform search automation of desired queries on Shodan. It gives a detailed inform of each result in the search, including vulnerabilities and exploits. It can be found in the [following GitHub repo](https://github.com/javirodriguezzz/Shodan-Browser).

### Files
- **main.py**  
Script which uses the Shodan API for search and obtain an inform from a query.
- **shodan-results.txt**  
Example results obtained by the script.

### Script usage

- **-q "Desired query"**  
Search a defined query.
- **-f "queries.txt"**  
Search from a file with a list of queries.

### Main method  
```python
def shodan_search(query, filepath):
    try:
        results = api.search(query)

        f = open(filepath, "w")
        for result in results['matches']:
            vuln_index = 0
            f.write("IP: " + result['ip_str'])
            f.write("\nProduct: " + str(result['http']['server']))
            # f.write("\nInfo: " + str(result['info']))
            f.write("\nOS: " + str(result['os']))
            f.write("\nOrganization: " + str(result['org']))
            f.write("\nUbication: " + result['location']['city'] + ", " + str(result['location']['longitude']) + "/" + str(result['location']['latitude']) + ", " + result['location']['country_name'])
            ports = api.host(result['ip_str'])
            f.write("\nOpen ports: " + str(ports))
            f.write("\nVulnerabilities: ")
            try:
                for vuln in result['vulns']:
                    f.write("\n  - " + vuln)
                    if vuln_index != 3:
                        exploits_api_url = "https://exploits.shodan.io/api/search?query={query}&key={api_key}"
                        exploit_query = exploits_api_url.format(query=vuln,
                                                                api_key=API_KEY)
                        response = requests.get(exploit_query)
                        try:
                            response_json = response.json()
                            if len(response_json['matches']) > 0:
                                for match in response_json['matches']:
                                    f.write('\nExploit: ' + match['description'])
                                    print('¡Exploit found! IP: ' + result['ip_str'])
                        except ValueError:
                            print('No exploits found for device ' + result['ip_str'] + "(" + vuln + ")")

                    vuln_index = vuln_index + 1
                    time.sleep(1)
            except KeyError:
                pass

            f.write("\n\n")
        f.close()
        print("Search finished for query: " + query)

    except shodan.APIError as e:
        print("Error: " + str(e))
```

### Output example
>IP: 185.X.X.X  
>Product: 360 web server, 792/71644  HTTP Server version 2.0 - TELDAT S.A.  
>OS: None  
>Organization: LTD HOSTPRO LAB  
>Ubication: Kyiv, 30.5238/50.45466, Ukraine  
>Open ports: 37215  
>Vulnerabilities:  
>  - **CVE-2019-1552**  
**Exploit:** OpenSSL has internal defaults for a directory tree where it can find a configuration file as well as 
     certificates used for verification in TLS. This directory is most commonly referred to as OPENSSLDIR, 
     and is configurable with the --prefix / --openssldir configuration options. For OpenSSL versions 1.1.0 and 
     1.1.1, the mingw configuration targets assume that resulting programs and libraries are installed in a Unix-like environment and the default prefix for program installation as well as for OPENSSLDIR should be '/usr/local'. However, mingw programs are Windows programs, and as such, find themselves looking at sub-directories of 'C:/usr/local', which may be world writable, which enables untrusted users to modify OpenSSL's default configuration, insert CA certificates, modify (or even replace) existing engine modules, etc. For OpenSSL 1.0.2, '/usr/local/ssl' is used as default for OPENSSLDIR on all Unix and Windows targets, including Visual C builds. However, some build instructions for the diverse Windows targets on 1.0.2 encourage you to specify your own --prefix. OpenSSL versions 1.1.1, 1.1.0 and 1.0.2 are affected by this issue. Due to the limited scope of affected deployments this has been assessed as low severity and therefore we are not creating new releases at this time. Fixed in OpenSSL 1.1.1d (Affected 1.1.1-1.1.1c). Fixed in OpenSSL 1.1.0l (Affected 1.1.0-1.1.0k). Fixed in OpenSSL 1.0.2t (Affected 1.0.2-1.0.2s).
>  - CVE-2018-0737  
**Exploit:** The OpenSSL RSA Key generation algorithm has been shown to be vulnerable to a cache timing side channel attack. An attacker with sufficient access to mount cache timing attacks during the RSA key generation process could recover the private key. Fixed in OpenSSL 1.1.0i-dev (Affected 1.1.0-1.1.0h). Fixed in OpenSSL 1.0.2p-dev (Affected 1.0.2b-1.0.2o).
>  - **CVE-2018-0734**
>  - **CVE-2018-0732**
>  - **CVE-2017-3737**  
**Exploit:** Node.js was affected by OpenSSL vulnerability CVE-2017-3737 in regards to the use of SSL_read() due to TLS handshake failure. The result was that an active network attacker could send application data to Node.js using the TLS or HTTP2 modules in a way that bypassed TLS authentication and encryption.  
**Exploit:** OpenSSL 1.0.2 (starting from version 1.0.2b) introduced an "error state" mechanism. The intent was that if a fatal error occurred during a handshake then OpenSSL would move into the error state and would immediately fail if you attempted to continue the handshake. This works as designed for the explicit handshake functions (SSL_do_handshake(), SSL_accept() and SSL_connect()), however due to a bug it does not work correctly if SSL_read() or SSL_write() is called directly. In that scenario, if the handshake fails then a fatal error will be returned in the initial function call. If SSL_read()/SSL_write() is subsequently called by the application for the same SSL object then it will succeed and the data is passed without being decrypted/encrypted directly from the SSL/TLS record layer. In order to exploit this issue an application bug would have to be present that resulted in a call to SSL_read()/SSL_write() being issued after having already received a fatal error. OpenSSL version 1.0.2b-1.0.2m are affected. Fixed in OpenSSL 1.0.2n. OpenSSL 1.1.0 is not affected.


#### Disclaimer: **The usage of this script it's only for academic purposes**.