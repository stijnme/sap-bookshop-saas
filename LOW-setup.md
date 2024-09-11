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

Test run:

```
cds watch --profile local-multitenancy
```

In another terminal session:

```
cds subscribe t1 --to http://localhost:4004 -u yves:
cds subscribe t2 --to http://localhost:4004 -u yves:
```

Remark: some port `4004`, since we don't run a seperate instance for the mtx module (as in the tutorial on port `4005`).
