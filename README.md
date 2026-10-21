# FusionPBX Pilot

Python library that automates [FusionPBX](https://www.fusionpbx.com) administration. FusionPBX has no complete REST API, so this library drives its web UI with Selenium WebDriver and exposes domains and extensions as plain Python objects (Page Object pattern): reading or assigning an attribute performs the corresponding action in the UI.

It is the engine behind [fusionpbx_pilot_api](https://github.com/marujofln/fusionpbx_pilot_api), a REST API for multi-tenant PBX provisioning. Published on [PyPI](https://pypi.org/project/fusionpbx-pilot/), licensed under GPL v3.

**Stack:** Python 3.10+, Selenium 4, pytest.

## Features

- Log in to FusionPBX and list, create, rename and delete **domains** (tenants)
- List, create, rename and delete **extensions** in a domain
- Read or set an extension's SIP password, voicemail status and voicemail e-mail
- Works with a local browser or a remote Selenium Grid
- Sample scripts in [`samples/`](samples/): customer report, bulk voicemail from CSV, bulk domain cleanup

## Installation

From PyPI:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install fusionpbx-pilot webdriver_manager
```

From source:

```bash
git clone https://github.com/marujofln/fusionpbx-pilot.git
cd fusionpbx-pilot
python3 -m venv .venv
source .venv/bin/activate
pip install -e . webdriver_manager
```

Firefox must be installed; `webdriver_manager` downloads the matching GeckoDriver.

## Quick start

```python
from selenium import webdriver
from selenium.webdriver.firefox.service import Service as FirefoxService
from webdriver_manager.firefox import GeckoDriverManager
from fusionpbx_pilot.page_objects import FusionPBX

driver = webdriver.Firefox(service=FirefoxService(GeckoDriverManager().install()))
f = FusionPBX(driver, 'https://pbx.example.com', '<login_user>', '<login_password>')

d = f.domain('tenant.example.com')               # raises DomainNotFound if missing
d = f.domain('new.example.com', create=True)     # creates the domain

e = d.extension('1001')                          # created if it does not exist
e.password = 'S3cure-Pass'
e.voicemail_enabled = True
e.voicemail_mail_to = 'user@example.com'
print(d.extensions.list())
```

## API

### Domains

| Operation | Code |
|---|---|
| List domains | `f.domains.list()` |
| Select a domain | `d = f.domain('<name>')` |
| Create a domain | `d = f.domain('<name>', create=True)` |
| Rename | `d.name = '<new_name>'` |
| Get UUID | `d.uuid` |
| Delete (needs the domain delete permission) | `del d.name` |

### Extensions

| Operation | Code |
|---|---|
| List extensions | `d.extensions.list()` |
| Select or create | `e = d.extension('<number>')` |
| Rename | `e.name = '<new_number>'` |
| SIP password | `e.password` / `e.password = '<new>'` |
| Voicemail enabled | `e.voicemail_enabled` / `e.voicemail_enabled = True` |
| Voicemail e-mail | `e.voicemail_mail_to` / `e.voicemail_mail_to = '<email>'` |
| Delete | `del d.extension.name` |

## Running the tests

The tests run against a real FusionPBX instance (they create and delete domains and extensions, so use a test server):

```bash
cp tests/.env_sample tests/.env   # set PYTEST_URL, PYTEST_USERNAME, PYTEST_PASSWORD
pip install pytest pytest-dependency python-dotenv faker
pytest tests/
```

## Contributing

Contributions are welcome: fork the project and open a pull request.
