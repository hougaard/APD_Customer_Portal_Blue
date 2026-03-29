# Customer Portal Build Progress

## STATUS: ✅ COMPLETE - All compiled successfully

## What was built:
All actions, templates, and scripts have been created. The portal is a dark/blue-themed Bulma-based customer portal for CRONUS (furniture).

### Actions Created:
| Action | Type | Template | Description |
|--------|------|----------|-------------|
| LOGIN | file | login | Login page - restyled to dark theme ✅ |
| LOGINCHECK | script | n/a | Login check script (was pre-existing) ✅ |
| DASHBOARD | file | dashboard | Main dashboard with stats, recent entries, quick actions ✅ |
| STATEMENT | file | statement | Customer ledger entries list ✅ |
| INVOICES | file | invoices | Posted invoices list with PDF download ✅ |
| CREDITMEMOS | file | creditmemos | Posted credit memos with PDF download ✅ |
| INVOICEPDF | report | n/a | Report 1306 (Sales Invoice) - Object No. set to 1306 ✅ |
| CREDITMEMOPDF | report | n/a | Report 1307 (Credit Memo) - Object No. set to 1307 ✅ |
| ORDERS | file | orders | Sales Quotes list ✅ |
| ORDERDETAIL | file | orderdetail | Edit quote - shows lines, add item form, delete line, browse items ✅ |
| NEWORDER | script | n/a | Creates new Sales Quote and redirects to ORDERDETAIL ✅ |
| ADDITEM | script | n/a | Adds item line to a quote ✅ |
| DELETEORDERLINE | script | n/a | Deletes a line from a quote ✅ |
| SHIPMENTS | file | shipments | Sales Shipment Header list ✅ |
| SHIPMENTDETAIL | file | shipmentdetail | Shipment lines detail view ✅ |
| PROJECTS | file | projects | Job/Project list for customer ✅ |
| PROJECTDETAIL | file | projectdetail | Project detail with Job Tasks ✅ |

### Templates Created:
- **portal-head** - Shared CSS (dark theme, Bulma, Font Awesome), burger menu JS
- **portal-header** - Navbar with navigation (Dashboard, Statement, Documents dropdown, Orders, Projects, Shipments), user info, logout
- **portal-footer** - Footer with company info from Company Information table
- **dashboard** - Stats cards, recent open entries (shows ALL open entries, no limit), quick actions, account info
- **statement** - Full ledger entry list with Open/Closed badges
- **invoices** - Invoice list with PDF download buttons
- **creditmemos** - Credit memo list with PDF download
- **orders** - Sales quotes list with View/Edit links
- **orderdetail** - Quote detail with add item form, item browser (expandable), line list with delete, total
- **shipments** - Shipment header list
- **shipmentdetail** - Shipment line detail (passes `shipno` parameter)
- **projects** - Job list with status badges, starting/ending dates, person responsible, View link
- **projectdetail** - Job detail header cards + Job Task table with indentation, task type badges (passes `jobno` parameter via action tag on Job table)
- **login** - Restyled to match dark theme

### Scripts (all compiled successfully):
- NEWORDER, ADDITEM, DELETEORDERLINE - all ✅

## AUTHENTICATION:
- **"Require Authentication" field** set to `1` on all actions EXCEPT LOGIN and LOGINCHECK
- **`<?auth?>` / `<?else?>` / `<?endif?>` blocks** removed from ALL page templates
- APD now handles authentication redirection at the action level instead of in-template checks

## BUG FIXES APPLIED:

### Session 3 - action() tag table name quoting:
The `action()` tag's second parameter (table name) must use double quotes, not single quotes, when the table name contains special characters like periods. Fixed in:
- `invoices` template: `<?action('INVOICEPDF',"Sales Invoice Header")?>`
- `creditmemos` template: `<?action('CREDITMEMOPDF',"Sales Cr.Memo Header")?>`
- `orders` template: `<?action('ORDERDETAIL',"Sales Header")?>`
- `shipments` template: `<?action('SHIPMENTDETAIL',"Sales Shipment Header")?>`

**Rule**: Always use double quotes for table names in `<?action()?>` tags.

### Session 4 - calc() not an expression function:
Dashboard had `<?set('dashcount',calc(Parm('dashcount')..'+1'))?>` which failed because `calc` is a **tag** not an expression function. You cannot use `calc()` inside expressions. Also, you cannot nest `<??>`  tags inside other `<??>` tags.

**Fix**: Removed the row-limiting counter entirely. The "Recent Open Entries" section now shows all open entries (which should be a manageable number). The loop is a simple `<?loop?>` / `<?endloop?>` without any counter logic.

**Rule**: `calc` is a TAG (`<?calc(expression)?>`), NOT an expression function. Don't use it inside `<?set?>` or other tag parameters. Also `<?loopno?>` cannot be nested inside `<?if?>` since you can't nest tags.

### Session 5 - no() and ping() not available in AL scripts:
The NEWORDER, ADDITEM, and DELETEORDERLINE scripts used `no()` and `ping()` which are **template expression functions**, NOT available in AL script context.

**Fix for no()**: Replaced with a lookup of the `APD User Hgd` table:
```al
APDUser.Get(currentuserid());
CustNo := APDUser."NAV Identification";
```

**Fix for ping() and action() in response redirect**: Template tags like `<?action()?>` and expression functions like `ping()` do NOT work inside AL scripts. Instead, build redirect URLs manually:
```al
response('redirect', '', '?action=ORDERDETAIL&docno=' + DocNo);
```

**Rule**: In AL scripts, you CANNOT use any template tags or template expression functions. Build URLs manually as `?action=ACTIONNAME&param=value`. Use `currentuserid()` and APD User Hgd table lookup for the BC number.

### Session 6 - Boolean field values:
When setting boolean fields on actions via `update_action`, use `1` (not `true` or `Yes`).
**Rule**: Boolean fields accept `1` for true, `0` for false.

### Session 7 - Projects/Jobs view added:
Added PROJECTS and PROJECTDETAIL actions with templates. Projects list shows Job table with status badges. Project detail shows Job header info cards + Job Task table with indentation and task type badges. Navigation updated in portal-header to include Projects link. The `jobno` parameter is passed via the `<?action('PROJECTDETAIL',Job)?>` tag (using the Job table primary key). The projectdetail template uses `<?get(Job,Parm('jobno'))?>` to load the specific job, and filters Job Task by `<?filter("Job Task","Job No.",Parm('jobno'))?>`.

### Session 8 - Option fields must use numeric index in <?if?> comparisons:
The `<?if?>` tag comparing Option fields (like Job.Status and Job Task."Job Task Type") must use the **numeric option index**, NOT the text value.

**Job.Status** option string: `Planning,Quote,Open,Completed` → `0,1,2,3`
- `<?if(Job,Status,'=','2')?>` for Open (NOT `'Open'`)
- `<?if(Job,Status,'=','0')?>` for Planning
- `<?if(Job,Status,'=','1')?>` for Quote
- `<?if(Job,Status,'=','3')?>` for Completed

**Job Task."Job Task Type"** option string: `Posting,Heading,Total,Begin-Total,End-Total` → `0,1,2,3,4`
- `<?if("Job Task","Job Task Type",'=','0')?>` for Posting
- `<?if("Job Task","Job Task Type",'=','1')?>` for Heading
- `<?if("Job Task","Job Task Type",'=','2')?>` for Total
- `<?if("Job Task","Job Task Type",'=','3')?>` for Begin-Total
- `<?if("Job Task","Job Task Type",'=','4')?>` for End-Total

**Rule**: For Option fields in `<?if?>` tags, ALWAYS use the numeric index (0-based from the OptionString), never the text name. The error "Cannot convert X to an integer" is the telltale sign of this mistake.

## Security Filters Needed:
The portal uses `no()` to filter data to the logged-in customer. The user needs to set up security filters on the APD user cards to restrict table access. Key tables that need security filters:
- Customer (18) - Filter on "No." = customer no
- Cust. Ledger Entry (21) - Filter on "Customer No."
- Sales Header (36) - Filter on "Sell-to Customer No."
- Sales Line (37) - via document relationship
- Sales Invoice Header (112) - Filter on "Sell-to Customer No."
- Sales Cr.Memo Header (114) - Filter on "Sell-to Customer No."
- Sales Shipment Header (110) - Filter on "Sell-to Customer No."
- Sales Shipment Line (111) - via document relationship
- Item (27) - May need to be open/unfiltered for browsing
- **Job (167) - Already has security filter set by user** ✅
- Job Task (1001) - May need security filter on "Job No." or rely on Job filter

## DESIGN NOTES:
- Dark theme: bg #0f172a, card bg #243349 (lighter blue), accent #3b82f6, text #cbd5e1/#f1f5f9
- Bulma CSS framework + Font Awesome icons
- Authentication is handled at the action level via "Require Authentication" field (not in-template `<?auth?>` checks)
- ORDERDETAIL passes `docno` parameter, SHIPMENTDETAIL passes `shipno` parameter, PROJECTDETAIL passes `jobno` parameter
- The WELCOME action and template should be IGNORED per instructions

### Session 9 - Card background lightened:
User requested content boxes have an even lighter shade of blue. Changed `--bg-card` from `#1e293b` to `#243349` (a noticeably lighter slate-blue) and `--bg-card-hover` from `#263548` to `#2c3d56`. This affects all `.dark-card` and `.stat-card` elements across the entire portal since the change is in the shared `portal-head` template.