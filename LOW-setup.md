# Init project

Create project:

```
cds init bookshop-saas --add samples
```

Install packages:

```
npm i
```

Create git repo:

```
git init
git add .
```

And commited first version of the project, this to see the config changes via git.

Test project:

```
cds watch
```

Hosted on `http://localhost:4004/`.
Following request in the browser shows 2 books:

```
http://localhost:4004/catalog/Books
```

# Add multitenancy

Add multi-tenant support:

```
cds add multitenancy --for production
```

This changed `package.json`:

```
+    "@sap/cds-mtxs": "^1",
     "express": "^4"
   },
   "devDependencies": {
@@ -14,5 +15,12 @@
   },
   "scripts": {
     "start": "cds run"
+  },
+  "cds": {
+    "requires": {
+      "[production]": {
+        "multitenancy": true
+      }
+    }
   }
 }
```

Install dependencies (for the added `@sap/cds-mtxs` package):

```
npm i
```

Test multitenancy locally:

```
cds add multitenancy --for local-multitenancy
```

This adds another config changes to `package.json`:

```
     "requires": {
       "[production]": {
         "multitenancy": true
+      },
+      "[local-multitenancy]": {
+        "multitenancy": true
       }
```

Remark: this is different from the [tutorial](https://cap.cloud.sap/docs/guides/multitenancy/), because we don't use a sidecar for the mtx module.

# Test run

```
cds watch --profile local-multitenancy
```

In another terminal session:

```
cds subscribe t1 --to http://localhost:4004 -u yves:
cds subscribe t2 --to http://localhost:4004 -u yves:
```

Remark: some port `4004`, since we don't run a seperate instance for the mtx module (as in the tutorial on port `4005`).

# Deploy SaaS to Cloud

Add config:

```
cds add hana,xsuaa,approuter --for production
npm i
```

This updates `package.json` again:

```
--- a/package.json
+++ b/package.json
@@ -8,7 +8,10 @@
   "dependencies": {
     "@sap/cds": "^6",
     "@sap/cds-mtxs": "^1",
-    "express": "^4"
+    "express": "^4",
+    "@sap/xssec": "^3",
+    "passport": "^0",
+    "hdb": "^0.19.0"
   },
   "devDependencies": {
     "sqlite3": "^5.0.4"
@@ -19,10 +22,22 @@
   "cds": {
     "requires": {
       "[production]": {
-        "multitenancy": true
+        "multitenancy": true,
+        "auth": {
+          "kind": "xsuaa"
+        },
+        "db": {
+          "kind": "hana-mt"
+        },
+        "approuter": {
+          "kind": "cloudfoundry"
+        }
       },
       "[local-multitenancy]": {
         "multitenancy": true
+      },
+      "db": {
+        "kind": "sqlite"
```

And adds following files and folders:

```
app/
db/src/
db/undeploy.json
xs-security.json
```

Add mta config:

```
cds add mta
```

This created an `mta.yaml` file, adjust following lines:

```
-        description: A simple CAP project.
-        category: 'Category'
+        description: Sample bookshop CAP project.
+        category: "Sample Applications"
```

Didn't bother to freeze `package-lock.json`

Install dependencies UI apps:

```
npm i --prefix app
```

Couldn't find any `package.json` files in the suggested locations by the tutorial:

```
app/browse
app/admin-books
```

# Build and deploy

```
mbt build -t gen --mtar mta_bookshop.tar
```

```
cf login --sso
cf deploy gen/mta_bookshop.tar
```
