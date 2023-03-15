title: "[CVE-2023-27569]-[CVE-2023-27570] Improper neutralization of SQL parameters in Profileo : Tracking et Conversions (eo_tags) module for PrestaShop"
categories: module
- Profileo.com
- TouchWeb.fr
meta: "CVE,PrestaShop,eo_tags"
severity: "high (9.8)"
* **CVE ID**: CVE-2023-27569 and CVE-2023-27570
* **Advisory source**: Friends-Of-Presta
* **Product**: eo_tags
* **Impacted release**: >= 1.2.0, < 1.4.19 (1.4.19 fixed the vulnerability)
* **Product author**: Profileo
* **Weakness**: [CWE-89](https://cwe.mitre.org/data/definitions/89.html)
* **Severity**: critical (9.8)
From version 1.2.0 published on Nov 17, 2017 to 1.4.18 published on Feb 21, 2023 (fixed in 1.4.19, published on Feb 28, 2023), an HTTP request can be forged with a compromized `_ga` cookie in order to exploit an insecure parameter in function `saveGanalyticsCookie()` and `gaParseCookie()`, which could lead to a SQL injection.

From version 1.2.0 published on Nov 17, 2017 to 1.2.19 published on Oct 22, 2019 (fixed in 1.3.0), an HTTP request can be forged with a compromised User-Agent or Referer in order to exploit insecure parameters in `trackReferrer()` function, which could lead to a SQL injection. As from 1.2.1, the code has been migrated to classes/EoTagsStats.php (`EoTagsStats::setNewGuest()`) and the vulnerability now requires Privileges (PR) and user interaction (UI) to be exploited, reducing the severity to 8.0.
* **Attack complexity**: low
* **User interaction**: none
* **Confidentiality**: high
* **Integrity**: high
* **Availability**: high
**Vector string**: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
* Obtain admin access
* Technical and personal data leaks

## Proof of concept

```bash
curl -v -H "User-agent:1','1');SELECT SLEEP(5);--" 'https://example.test/'
curl -v -H "User-agent:1','1');SELECT SLEEP(5);--" 'https://example.test/?force_eo_tags_tracking=1'
curl -v --cookie "PrestaShop-xyz" --cookie "_ga=1.1.1','1')%3BSELECT SLEEP(5)%3B--" 'https://example.test/order'
```
## Patch 
If present in `eo_tags.php`
--- a/eo_tags.php
+++ b/eo_tags.php
@@ -1495,8 +1495,8 @@ class Eo_Tags extends Module
                 $old_cid = $this->getAnalyticsCID($this->context->cart->id);
                 $data = array(
                     'id_cart' => $this->context->cart->id,
-                    'cid'     => $cid,
-                    'cookie'  => serialize($_COOKIE['_ga']),
+                    'cid'     => pSQL($cid),
+                    'cookie'  => pSQL(serialize($_COOKIE['_ga'])),
                 );
                 if (!$old_cid) {
                     Db::getInstance()->insert('eo_tags_ga_cookie', $data);
```

```diff
--- a/eo_tags.php
+++ b/eo_tags.php
@@ -1356,11 +1356,11 @@ class Eo_Tags extends Module
                 ';
                 if ($referral = Db::getInstance()->getRow($sql2)) {
                     $data = array(
-                        'id_guest'     => $cookie->id_guest,
+                        'id_guest'     => (int)$cookie->id_guest,^M
                         'ip_address'   => $referral['ip_address'],
-                        'http_referer' => $referral['http_referer'],
-                        'request_uri'  => $referral['request_uri'],
-                        'user_agent'   => $user_agent,
+                        'http_referer' => pSQL($referral['http_referer']),^M
+                        'request_uri'  => pSQL($referral['request_uri']),^M
+                        'user_agent'   => pSQL($user_agent),^M
                         'date_add'     => $referral['date_add'],
                     );
@@ -1397,11 +1397,11 @@ class Eo_Tags extends Module
             $request_uri = substr($request_uri, 0, 255);
             $data = array(
-                'id_guest'     => $cookie->id_guest,
+                'id_guest'     => (int)$cookie->id_guest,^M
                 'ip_address'   => $ip_address,
-                'http_referer' => $http_referer,
-                'request_uri'  => $request_uri,
-                'user_agent'   => $user_agent,
+                'http_referer' => pSQL($http_referer),^M
+                'request_uri'  => pSQL($request_uri),^M
+                'user_agent'   => pSQL($user_agent),^M
                 'date_add'     => date('Y-m-d H:i:s'),
             );
         }
If present in `classes/EoTagsStats.php` `EoTagsStats::setNewGuest()`
```diff
--- a/classes/EoTagsStats.php
+++ b/classes/EoTagsStats.php
@@ -26,7 +26,7 @@ class EoTagsStats {
         $data = array(
             'id_customer' => $id_customer,
             'ip_address'  => $ip_address,
-            'user_agent'  => $user_agent,
+            'user_agent'  => pSQL($user_agent),^M
             'date_add'    => date('Y-m-d H:i:s'),
         );
```

Profileo thanks TouchWeb.fr for its help discovering the vulnerability.

## Timeline
| Date | Action |
| -- | -- |
| 2023-02-24 | Discovery of the vulnerability by TouchWeb.fr |
| 2023-02-25 | Vulnerability confirmed by Profileo |
| 2023-02-28 | Patch created by Profileo and release of version 1.4.19 fixing the issue |
| 2023-03-01 | Patch released to customers |
| 2023-03-15 | Publication on security.profileo.com |
* [Profileo](https://www.profileo.com/fr/)
* [TouchWeb.fr](https://www.touchweb.fr/)
* [National Vulnerability Database](https://nvd.nist.gov/vuln/detail/CVE-YYYY-XXXX)