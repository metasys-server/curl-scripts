# Curl Scripts

> **Note** The scripts and documentation in this repository were written against version 2 of the *Metasys*® REST API.

The scripts in this directory show the basics of accessing
the *Metasys*® Server API from the command line using `curl`.

These scripts have the following dependencies:

* bash (or zsh) - These scripts assume a "unix-like" shell. Tested with bash and zsh.
* curl - standard on mac, unix and linux. Available for Windows
  thru Windows Subsystem for Linux (WSL), cygwin or chocolatey

    See the [man page](https://curl.haxx.se/docs/manpage.html) for details on how to use curl
* [jq](https://stedolan.github.io/jq/download/) - A command line library for processing  JSON. Take a look at the [tutorial](https://stedolan.github.io/jq/tutorial/) and [manual](https://stedolan.github.io/jq/manual/) and you'll see it's a very nice powerful tool.

The following scripts exist

* devices - logs in and retrieves the first pages of network devices.

* test-drive - logs in and makes one call to each of the major endpoints.

They are located in their own respective folders.

## Getting Started with Curl

You can make requests of *Metasys*® right from the command line given you have `curl`. The `jq` utility
will also come in handy for processing the JSON responses you get from the server.

This example will show you how to get an access token and fetch the first page of devices.
*Be sure to replace `hostname` with the actual name or ip address of your server.*

### Login

To get a token you must pass some credentials. The easiest way to do this is to create a file
that contains your username and password and then use `curl` to pass this file to the login endpoint.

**Warning** Be sure to protect access to this file since it contains your credentials. While this
method works for educational purposes it is not recommended for a production site. Consider using a credential store to securely store this information.

```json
{
  "username": "johnsmith",
  "password": "xj38sj9##Abo-y"
}
```

Assuming this file was named `credentials.json` you can get an access token with the following command.
(Note: The accessToken shown has been truncated for sake of space.)

```bash
$ curl --data-binary @credentials.json -H "Content-Type: application/json" https://hostname/api/v2/login
{"accessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJ...","expires":"2018-09-17T19:38:58Z"}
```

**Warning** Be sure to protect your access tokens as they grant anyone who has one access to your *Metasys*® site.

*Explanation* We have used the `--data-binary` option to specify a file that contains the payload we need to send to the server. The `-H` option specifies a `Content-Type` header
which is required so the server knows how to interpret the data we send it. The server responds by returning
a JSON document with two fields: `accessToken` and `expires`. The `expires` field lets us know when this token expires.

**Note** If you do not see a valid response but instead see the following (or something similar), that means there is an issue with your SSL cert. See [Certificate Issues](#certificate-issues) for a discussion of possible solutions.

> curl: (60) SSL certificate problem: self signed certificate
> More details here: https://curl.haxx.se/docs/sslcerts.html
>
> curl failed to verify the legitimacy of the server and therefore could not
> establish a secure connection to it. To learn more about this situation and
> how to fix it, please visit the web page mentioned above.

### Use Access Token to Access a Resource

Here's how you might use this access token to access one of the *Metasys*® APIs (copy and paste the token from the previous login):

```bash
curl -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJ..." https://hostname/api/v2/networkDevices
```

We are using the `-H` option to specify an `Authorization` header which is used to pass our access token.
The content of the header is the word `Bearer` followed by a space and then our access token that we got in the previous step.

**Note** If you are using cygwin in the old windows terminal your copy/paste may fail as it inserts line breaks to format to the screen. Skip ahead to the next section or paste to a text file and manually remove the line breaks before using the token.

### Putting it all together

To avoid having to copy/paste the access token we can use `jq` to extract the `accessToken` out of the
login request and store it in a variable. This variable can then be used to pass the `accessToken`
to the next call. (In the following example, the response payload from `networkDevices` has been truncated
for space.)

**Warning** Be sure to protect your access token. When you are done with this example you should `unset TOKEN` or close your console to minimize the chance of leaking it.

```json
$ TOKEN=$(curl --data-binary @credentials.json -H "Content-Type: application/json" https://hostname/api/v2/login | jq -r .accessToken)
$ curl -H "Authorization: Bearer $TOKEN" https://hostname/api/v2/networkDevices
{"total":12,"next":null,"previous":null,"items":[{"id":"f55a7799-ec10-5361-8569-04258bdd8070","itemReference":"Test-pc1:Test-NAE5510","name":"Test-NAE5510","type":"/enumSets/508/members/185","description":"Auth Cat Fire","firmwareVersion":"8.0.0.0449","category":"/enumSets/33/members/1","timeZone":"/enumSets/576/members/53","self":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070","parent":null,"networkDevices":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/networkDevices","equipment":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/equipment","spaces":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/spaces","objects":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/objects","attributes":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/attributes","alarms":"/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/alarms"},...]}
$ unset TOKEN
```

Let's break down what is happening in the first call. Like before we are using curl to call the
`/login` endpoint. We then pipe the result to `jq`. We tell `jq` to extract out just the access token
by using the filter `accessToken`. The `-r` option tells `jq` to return raw output. Without this option
it would return our token with double quotes around it which is not what we want.

In the second curl call we are using variable expansion to pass in the token we got in the first step.

## Getting a Little Fancier

The `login` endpoint accepts form encoded data as well. This makes it a little easier to use since it
doesn't require the use of a separate file.

```sh
TOKEN=$(curl -d username=yourusername -d password=yourpassword https://hostname/api/v2/login | jq -r .accessToken)
```

**Warning** Take care when writing scripts that include a password. It would be better to actually read the password from a credential store. Also this may not work if your password has special chars that the shell treats special (like `!`)

You'll notice that the responses that come back from the server do not have any extra whitespace. If you would like
them "pretty printed" you can use `jq` for that by simply passing it the identity filter `.`.

```sh
$ curl -H "Authorization: Bearer $TOKEN" https://hostname/api/v2/networkDevices | jq .
{
  "total": 12,
  "next": null,
  "previous": null,
  "items": [
    {
      "id": "f55a7799-ec10-5361-8569-04258bdd8070",
      "itemReference": "Test-pc1:Test-NAE5510",
      "name": "Test-NAE5510",
      "type": "/enumSets/508/members/185",
      "description": "Auth Cat Fire",
      "firmwareVersion": "8.0.0.0449",
      "category": "/enumSets/33/members/1",
      "timeZone": "/enumSets/576/members/53",
      "self": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070",
      "parent": null,
      "networkDevices": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/networkDevices",
      "equipment": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/equipment",
      "spaces": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/spaces",
      "objects": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/objects",
      "attributes": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/attributes",
      "alarms": "/networkDevices/f55a7799-ec10-5361-8569-04258bdd8070/alarms"
    },
...
}
$ unset TOKEN
```

## Certificate Issues

Curl checks the certificates of your server. If the certificate is not trusted then the curl operations will fail. Here are some workarounds. See [SSL Certification Verification](https://curl.haxx.se/docs/sslcerts.html) for more information.

1. Manually add the certificate to your local keystore and mark it as trusted
2. Export the certificate in pem format and pass it on the command line.
3. Use the `--insecure` option. **Note:** this is not recommended for production environments.

There are examples of how to use the last 2 options in the READMEs of the subfolders: [devices](./devices/README.md#certificate-issues), [test-drive](./test-drive/README.md#certificate-issues).
