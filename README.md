# Databricks : Connecting-to-Storage-Account

<p><strong><u>Legacy Method</u></strong></p>
<p><strong>Methods that were existing prior to existence of Unity Catalog</strong></p>
<p><strong><em>Access via Account Key</em></strong></p>
<p>just a string that give full access to a Storage Account(DataLake)</p>
<p><strong><em>&nbsp;Access via Service Principal </em></strong></p>
<p>Simply an account explicitly created in Azure that you can later use to grant some permissions.</p>
<p>You will create a Service Principal, and grant it access to DataLake. For this you can either use RBAC or ACL</p>


## Access via Service Principal

<p><strong>Go to Azure Portal -&gt; Microsoft Entra ID</strong></p>
<p>Inside the Microsoft Entra ID you can create Service Principles</p>

![image](https://github.com/user-attachments/assets/80a490ae-d0fb-4772-a7d1-e49da2dc748e)


<p><strong>Go to App Registration</strong></p>

![image](https://github.com/user-attachments/assets/31f27bbb-3e4f-4a91-aab3-226fd2219e32)

<p><strong>Then click &ldquo;New registration&rdquo;</strong></p>

![image](https://github.com/user-attachments/assets/b442904b-3318-4833-bee3-0d70b34b67f1)

<p><strong>Give name to it</strong></p>

<p><strong>And hit &ldquo;Register&rdquo;</strong></p>


![image](https://github.com/user-attachments/assets/0a3e08f1-2c3f-4b6a-afb9-3e6069552034)

<p>You can create several types of Access with this Service Principal. But here we are using Secrets</p>


![image](https://github.com/user-attachments/assets/21f53228-a3b6-4840-8777-4ee6dcff19d6)

<p><strong>Create a new Client Secret</strong></p>


![image](https://github.com/user-attachments/assets/f39c798f-9310-4bff-8690-29db831440ad)

<p><strong>Give it a proper description</strong></p>

![image](https://github.com/user-attachments/assets/ab496855-37bd-4603-b8cf-c979636329c1)


<p><strong>You will get some value that can be used later as form of Secret</strong></p>

![image](https://github.com/user-attachments/assets/3880eaa4-9956-4c18-94d1-a0311bc0f356)


## So, far Service Principal is created. Next part is granting this Service Principal RBAC to Data Lake

<p><strong>Storage Account Level</strong></p>
<p><strong>Go to Access Control -&gt; Role Assignment </strong></p>
<p>You will see all the roles under this. If there is already some group present in there which already had same set of permissions that you want your newly created Service Principal to have(Storage Blob Contributor Role in this case) then you simply add your Service Principle to that group. To do this you have to go to</p>

<p><strong>Microsoft Entra ID -&gt; Groups -&gt; find your group -&gt; add a new member(Service Principal in this case) to the group</strong></p>

<p><strong>Bonus:&nbsp;</strong>Group creation steps</p>

![image](https://github.com/user-attachments/assets/ff79fa4e-e67a-42cc-992e-624a261cea65)

![image](https://github.com/user-attachments/assets/ccc2ee4d-4231-4a43-b21d-825827a4e1b3)

![image](https://github.com/user-attachments/assets/0578cd87-1ebe-4f7f-974f-bd9fc40e8158)


![image](https://github.com/user-attachments/assets/8b84d486-8231-4268-9ddf-cc5274884b41)



![image](https://github.com/user-attachments/assets/deb33564-9811-4e68-bcee-ccb037543056)

![image](https://github.com/user-attachments/assets/234b80e3-d2f8-463c-bddc-46236826fffd)

![image](https://github.com/user-attachments/assets/a435c126-eb62-4543-b9cc-3cc66583d801)

![image](https://github.com/user-attachments/assets/0ab5cecc-d68b-41e9-9ac7-b64d5b2ea1f1)

![image](https://github.com/user-attachments/assets/17b93254-43e4-45d8-9168-cc5d431d549b)

![image](https://github.com/user-attachments/assets/d5a482ae-fcb2-49d4-bf66-2267192a8631)

![image](https://github.com/user-attachments/assets/dd41867b-8864-4839-8e49-70f39d45efa2)


<p>Now if you go to Databricks and create a new notebook and then try to connect to the DataLake, the connection should fail because you haven&rsquo;t configured the Databricks yet.</p>

<div class="heading-wrapper" data-heading-level="h4">
<h4 id="azureserviceprincipal" class="heading-anchor">Azure&nbsp;service&nbsp;principal</h4>
</div>
<p>Use the following format to set the cluster Spark configuration:</p>

```python
service_credential = dbutils.secrets.get(scope="<secret-scope>",key="<service-credential-key>")

spark.conf.set("fs.azure.account.auth.type.<storage-account>.dfs.core.windows.net", "OAuth")
spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account>.dfs.core.windows.net", "<application-id>")
spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account>.dfs.core.windows.net", service_credential)
spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account>.dfs.core.windows.net", "https://login.microsoftonline.com/<directory-id>/oauth2/token")
```

<div class="heading-wrapper" data-heading-level="h4">
<p>Replace</p>
<ul>
<li><code>&lt;secret-scope&gt;</code>&nbsp;with the Databricks secret scope name.</li>
<li><code>&lt;service-credential-key&gt;</code>&nbsp;with the name of the key containing the client secret.</li>
<li><code>&lt;storage-account&gt;</code>&nbsp;with the name of the Azure storage account.</li>
<li><code>&lt;application-id&gt;</code>&nbsp;with the&nbsp;<strong>Application (client) ID</strong>&nbsp;for the Microsoft Entra ID application.</li>
<li><code>&lt;directory-id&gt;</code>&nbsp;with the&nbsp;<strong>Directory (tenant) ID</strong>&nbsp;for the Microsoft Entra ID application.</li>
</ul>
</div>

![image](https://github.com/user-attachments/assets/3e831479-58b3-4022-ad5e-c4a600ce4a4c)

