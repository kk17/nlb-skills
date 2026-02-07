---
name: nlb
description: check loans and search resources from the National Library Board of Singapore
homepage: https://www.nlb.gov.sg
metadata:
  { "clawdbot": { "emoji": "📚", "requires": { "bins": [] }, "install": [] } }
---

# NLB Skill

## Login to NLB

NLB uses a multi-domain authentication system with CAS (Central Authentication Service).

### Automated Login (API-based)

The login process involves these steps:

1. **Get login page and extract execution token:**
   ```bash
   EXECUTION=$(curl -s "https://signin.nlb.gov.sg/authenticate/login" | grep -oP 'name="execution"\s+value="[^"]+"' | cut -d'"' -f4 | head -1)
   ```

2. **Login with credentials:**
   ```bash
   curl -s -c /tmp/cookies.txt -L -X POST "https://signin.nlb.gov.sg/authenticate/login" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
     --data-urlencode "username=<username>" \
     --data-urlencode "password=<password>" \
     --data-urlencode "_eventId=submit" \
     --data-urlencode "submit=CONTINUE" \
     --data-urlencode "execution=$EXECUTION"
   ```

3. **Access myLibrary endpoints with saved cookies:**
   The cookies are saved in `/tmp/cookies.txt` and can be used for subsequent requests.

**Important Notes:**
- NLB uses multiple domains: `signin.nlb.gov.sg` (auth) and `www.nlb.gov.sg` (main site)
- Cookies must be properly captured and sent for authenticated requests
- Use `--data-urlencode` for form data to handle special characters
- Session cookies include: `AWSALB`, `AWSALBCORS`, `TGC`, `JSESSIONID`

### Manual Login

1. Open https://signin.nlb.gov.sg/authenticate/login
2. Use user myLibrary username and password to login
3. Alternatively, use Singpass or NLB Mobile app for authentication

## Check Loans(requires login)

### Automated (API)

**Current Loans:**
```bash
curl -s 'https://www.nlb.gov.sg/mylibrary/Loans/GetLoans?FilterBy=checkouts-active&_rand=test' \
  -H 'cookie: <cookies from login>' \
  -H 'Referer: https://www.nlb.gov.sg/mylibrary/Loans' \
  -H 'X-Requested-With: XMLHttpRequest'
```

**Overdue Loans:**
```bash
curl -s 'https://www.nlb.gov.sg/mylibrary/Loans/GetLoans?FilterBy=checkouts-overdue&_rand=test' \
  -H 'cookie: <cookies from login>' \
  -H 'Referer: https://www.nlb.gov.sg/mylibrary/Loans' \
  -H 'X-Requested-With: XMLHttpRequest'
```

**Response Format:**
Returns JSON with `view` containing HTML table of loans, and metadata:
- `TotalRecords`: Number of loans
- `TotalCount`: Total count
- `NextRecordsOffset`: For pagination
- `Error`: Any error message

### Manual

1. Open https://www.nlb.gov.sg/mylibrary/loans
2. Check "Current" Tab to see borrowed items and due dates
3. Check "Overdue" Tab to see past borrowed items

## Renew Loans(requires login)

```bash
# Get list of item numbers from current loans API response
# Then renew specific items:
curl -s -X POST 'https://www.nlb.gov.sg/mylibrary/Loans/RenewLoans' \
  -H 'cookie: <cookies from login>' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Referer: https://www.nlb.gov.sg/mylibrary/Loans' \
  -H 'X-Requested-With: XMLHttpRequest' \
  --data-urlencode "itemNo=<barcode>" \
  --data-urlencode "__RequestVerificationToken=<token>"
```

## Check Recommendations(requires login)

1. Open https://www.nlb.gov.sg/mylibrary/Home

## Search Resources

1. URL Encode the search query
2. Open search result page: https://catalogue.nlb.gov.sg/search?query={url_encoded_query}
   Optional URL parameter filters:
   - &universalLimiterIds=at_library (to filter to only items available at libraries)
   - &pageNum=0 (to specify page number, starting from 0)
   - &viewType=grid (to view results in grid format)
   - &materialTypeIds=1 (to filter to books only)
   - &collectionIds={collection_ids} (to filter by specific collections, see below for details)
   - &locationIds={location_ids} (to filter by specific libraries, see below for details)
   - languageIds={language_ids} (to filter by specific languages, chi: Chinese, eng: English, mal: Malay, tam: Tamil)

### Collection Id Mappings:
| Collection Name                      | Collection Id |
| ------------------------------------ | ------------- |
| Early Literacy 4-6                   | 7             |
| Junior Picture Book                  | 11            |
| Junior                               | 9             |
| Early Literacy Accessible Collection | 55            |
| Junior Simple Fiction                | 12            |
| Junior Accessible Collection         | 8             |
| Adult                                | 3             |

### Location Id Mappings:
| Library Name      | Location ID |
| ----------------- | ----------- |
| Toa Payoh Library | 23          |
| Bishan Library    | 6           |
| Central Library   | 29          |


### Example

For search query "BookLife readers", filtering to items book only, available at Toa Payoh Library, in Junior Picture Book and Junior collections, viewing first page in grid format:
https://catalogue.nlb.gov.sg/search?query=BookLife%20readers&pageNum=0&locationIds=23&universalLimiterIds=at_library&collectionIds=11,9,7&viewType=grid&materialTypeIds=1

## Open Search Card Page
1. Click the search result link to open the search result page in browser
Search Card Page Example:
https://catalogue.nlb.gov.sg/search/card?id=127ebe36-bad7-566c-8d81-3a32379254ad&entityType=FormatGroup

## Key API Endpoints

### Authentication
- **Login Page:** https://signin.nlb.gov.sg/authenticate/login
- **Login POST:** https://signin.nlb.gov.sg/authenticate/login
- **Home Page:** https://www.nlb.gov.sg

### MyLibrary APIs
- **Current Loans:** https://www.nlb.gov.sg/mylibrary/Loans/GetLoans?FilterBy=checkouts-active
- **Overdue Loans:** https://www.nlb.gov.sg/mylibrary/Loans/GetLoans?FilterBy=checkouts-overdue
- **Loans Page:** https://www.nlb.gov.sg/mylibrary/Loans
- **Renew Loans:** https://www.nlb.gov.sg/mylibrary/Loans/RenewLoans
- **Loan History:** https://www.nlb.gov.sg/mylibrary/Loans/History
- **Reservations:** https://www.nlb.gov.sg/mylibrary/Reserves
- **Reservation History:** https://www.nlb.gov.sg/mylibrary/Reserves/History
- **eBook Loans:** https://www.nlb.gov.sg/mylibrary/Ebooks/Loans
- **eBook Reservations:** https://www.nlb.gov.sg/mylibrary/Ebooks/Reserves
- **Account:** https://www.nlb.gov.sg/mylibrary/Account
- **Home:** https://www.nlb.gov.sg/mylibrary/Home

### Catalog Search
- **Search:** https://catalogue.nlb.gov.sg/search
- **Item Details:** https://eservice.nlb.gov.sg/redir/itemdetails?bid=<id>
