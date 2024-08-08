

## Description

LinkedInDumper is a Python 3 script that dumps employee data from the LinkedIn social networking platform.

The results contain firstname, lastname, position (title), location and a user's profile link. Only 2 API calls are required to retrieve all employees if the company does not have more than 10 employees. Otherwise, we have to paginate through the API results. With the `--email-format` CLI flag one can define a Python string format to auto generate email addresses based on the retrieved first and last name.

## Requirements

LinkedInDumper talks with the unofficial LinkedIn Voyager API, which requires authentication. Therefore, you must have a valid LinkedIn user account. To keep it simple, LinkedInDumper just expects a cookie value provided by you. Doing it this way, even 2FA protected accounts are supported. Furthermore, you are tasked to provide a LinkedIn company URL to dump employees from.

#### Retrieving LinkedIn Cookie

1. Sign into www.linkedin.com and retrieve your ``li_at`` session cookie value e.g. via developer tools
2. Specify the cookie value either persistently in the python script's variable ``li_at`` or temporarily during runtime via the CLI flag ``--cookie``

#### Retrieving LinkedIn Company URL

1. Search your target company on Google Search or directly on LinkedIn
2. The LinkedIn company URL should look something like this: https://www.linkedin.com/company/apple

## Usage

````
usage: linkedindumper.py [-h] --url <linkedin-url> [--cookie <cookie>] [--quiet] [--include-private-profiles] [--jitter] [--email-format EMAIL_FORMAT]

options:
  -h, --help            show this help message and exit
  --url <linkedin-url>  A LinkedIn company url - https://www.linkedin.com/company/<company>
  --cookie <cookie>     LinkedIn 'li_at' session cookie
  --quiet               Show employee results only
  --include-private-profiles
                        Show private accounts too
  --jitter              Add a random jitter to HTTP requests to bypass rate limiting
  --email-format EMAIL_FORMAT
                        Python string format for emails; for example:
                        --email-format '{0}.{1}@example.com' --> john.doe@example.com
                        --email-format '{0[0]}.{1}@example.com' --> j.doe@example.com
                        --email-format '{1}@example.com' --> doe@example.com
                        --email-format '{0}@example.com' --> john@example.com
                        --email-format '{0[0]}{1[0]}@example.com' --> jd@example.com
````

### Example 1 - Docker Run

````
docker run --rm l4rm4nd/linkedindumper:latest --url 'https://www.linkedin.com/company/apple' --cookie '<cookie>' --email-format '{0}.{1}@apple.de'
````

### Example 2 - Native Python

````
# install dependencies
pip install -r requirements.txt

python3 linkedindumper.py --url 'https://www.linkedin.com/company/apple' --cookie '<cookie>' --email-format '{0}.{1}@apple.de'
````

## Outputs

The script will return employee data as semi-colon separated values (like CSV):

[i] Company Name: apple
[i] Company X-ID: 162479
[i] LN Employees: 1000 employees found
[i] Dumping Date: 17/10/2022 13:55:06
[i] Email Format: {0}.{1}@apple.de

Firstname;Lastname;Email;Position;Gender;Location;Profile
Katrin;Honauer;katrin.honauer@apple.de;Software Engineer at Apple;N/A;Heidelberg;https://www.linkedin.com/in/katrin-honauer
Raymond;Chen;raymond.chen@apple.de;Recruiting at Apple;N/A;Austin, Texas Metropolitan Area;https://www.linkedin.com/in/raytherecruiter

[i] Successfully crawled 2 unique apple employee(s). Hurray ^_-
````

## Limitations

LinkedIn will allow only the first 1,000 search results to be returned when harvesting contact information. You may also need a LinkedIn premium account when you reached the maximum allowed queries for visiting profiles with your freemium LinkedIn account.

Furthermore, not all employee profiles are public. The results vary depending on your used LinkedIn account and whether you are befriended with some employees of the company to crawl or not. Therefore, it is sometimes not possible to retrieve the firstname, lastname and profile url of some employee accounts. The script will not display such profiles, as they contain default values such as "LinkedIn" as firstname and "Member" in the lastname. If you want to include such private profiles, please use the CLI flag ``--include-private-profiles``. Although some accounts may be private, we can obtain the position (title) as well as the location of such accounts. Only firstname, lastname and profile URL are hidden for private LinkedIn accounts.

Finally, LinkedIn users are free to name their profile. An account name can therefore consist of various things such as saluations, abbreviations, emojis, middle names etc. I tried my best to remove some nonsense. However, this is not a complete solution to the general problem. Note that we are not using the official LinkedIn API. This script gathers information from the "unofficial" Voyager API.

If you perceive an error message like `maximum request exceeded`, try the `--jitter` CLI parameter to introduce some delays for the automated API requests. This may be able to bypass the implemented detections or rate limiting by LinkedIn.

