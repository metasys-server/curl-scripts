# Test Drive

This bash script `test-drive` demonstrates how to call many of the endpoints supported by *Metasys*® Server REST API (version 2).

These scripts have the following dependencies:

* bash - Tested on mac, linux, and windows (using Git Bash on Win 7, Ubuntu on Win 10)
* curl - standard on mac and linux. Available for Windows thru cygwin
* [jq](https://stedolan.github.io/jq/download/) - A command-line JSON processor.
  This is used to process the json results from the server.

## Running the scripts

The script requires a username, password and hostname.
You will be prompted for these every time you run the script.

```shell
$ ./test-drive
username: johnsmith
password:
host: adx-server

Make login request

Getting first page of network devices:
  https://adx-server/api/v2/networkDevices

Getting first page of objects for the first device:
  https://adx-server/api/v2/objects/bd240c93-9fd8-5590-ae79-89b51fdf21ff/objects

Getting the default view of the first object in the list with schema
  https://adx-server/api/v2/objects/403c7f8b-c6f3-5061-b10d-bcab0beca0e2?includeSchema=true

Getting the first page of alarms for the first device
  https://adx-server/api/v2/objects/bd240c93-9fd8-5590-ae79-89b51fdf21ff/alarms

Getting the first page of audits for the first device
  https://adx-server/api/v2/objects/bd240c93-9fd8-5590-ae79-89b51fdf21ff/audits

Getting the first page of equipment
  https://adx-server/api/v2/equipment

Getting the first page of spaces
  https://adx-server/api/v2/spaces

Getting the first page of alarms for the entire site
  https://adx-server/api/v2/alarms

Getting the first page of audts for the entire site
  https://adx-server/api/v2/audits

Getting the first 1000 enum sets
  https://adx-server/api/v2/enumSets?pageSize=1000
```

## Output

The results of each call are saved to a file in the `output` directory.

## Protect The Access Token

You must protect any access token you get from the Metasys Server. Anyone with access to this token
can impersonate you for the life of the session associated with that token.

This program stores an access token in two files in the `output/` directory (`output/login-result.json` and `output/access_token.txt`).

> **This is solely for
educational purposes to allow a developer to inspect the results. This is not a recommended practice for a production system. You are responsible for protecting your access tokens.**

## Certificate Issues

Curl checks the certificates of your server. If the certificate is not trusted then the curl operations will fail. Here are some workarounds. See [SSL Certification Verification](https://curl.haxx.se/docs/sslcerts.html) for more information.

1. Manually add the certificate to your local keystore and mark it as trusted
2. Export the certificate in pem format and pass it on the command line. The following example assumes you have done this and named the file `server-certificate.pem`

    ```shell
    ./test-drive --ca-cert server-certificate.pem
    ```
3. Use the `--insecure` option. **Note:** this is not recommended for production environments.

    ```shell
    ./test-drive --insecure
    ```

## Further Resources

* curl
* jq
* bash
