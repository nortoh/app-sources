## WARNING

Before uploading any new Magento versions here. Ensure that you have patched `app/etc/di.xml` with the following change:

```
--- app/etc/di.xml.orig	2023-03-08 15:13:21.000000000 -0500
+++ app/etc/di.xml	2023-03-08 15:13:46.000000000 -0500
@@ -1853,7 +1853,7 @@
             <argument name="supportedVersionPatterns" xsi:type="array">
                 <item name="MySQL-8" xsi:type="string">^8\.0\.</item>
                 <item name="MySQL-5.7" xsi:type="string">^5\.7\.</item>
-                <item name="MariaDB-(10.2-10.4)" xsi:type="string">^10\.[2-4]\.</item>
+                <item name="MariaDB-(10.2-10.5)" xsi:type="string">^10\.[2-5]\.</item>
             </argument>
         </arguments>
     </type>
```
