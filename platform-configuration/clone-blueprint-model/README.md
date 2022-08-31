This guide covers the basics of creating a new Platform Blueprint and Platform Model based off an existing one. The use case is the user wants to create a parallel Oracle FMW domain.


## Infrastructure Provider
1. Go to Infrastructure > **Infrastructure Providers**
2. Create the necessary provider and/or hosts. For example, if you are:
	1. **Reusing existing infrastructure** then no action is required.
	2. **Using new infrastructure** then select your provider (or create a new provider) and create new hosts<br><br>

**IMPORTANT**
1. There is a known bug with Compute Logical Definition where only OL6 should be selected. If any other option is selected you will not see the hosts when creating a new Platform Model.
2. There is no impact to selecting OL6. This option is only for visual purposes and features based off this were not yet implemented in Myst.<br> ![[img/infrastructure_host.png]]


## Platform Blueprint
#### Creating a new Blueprint from a Template
1. Go to **Platform Blueprints** and click your blueprint
2. Click Actions > **Save as Template** <br> ![[img/template-save.png]]
3. Fill in the new template information

#### Updating the Template
1. Go to **Platform Templates** and click your template
2. Here you can update your template as required. For example, you can update:
	1. Middleware Settings > Catalog Version
	2. Middleware Settings > Home Directory
	3. WebLogic Domain Configuration > Name
3. *(OPTIONAL)* To permanently disable editing on the template you can click Actions > **Publish** <br> ![[img/template-publish.png]]


#### Create a Blueprint
1. Go to Platform Templates > Actions > **Generate Blueprint** <br> ![[img/template-generate-blueprint.png]]
2. Fill in the new blueprint information and click **Next** and **Finish**
3. You are taken directly to the blueprint where you can **Save & Commit** the initial revision
4. You are now ready to create a model from the blueprint


## Platform Model
#### Create a Model
1. Click Platform Model > **Create New**
2. Select the **platform blueprint** created earlier
3. Fill in the new model information and click **Next** until you finally reach the **Finish** button
4. Verify your model and edit as required and **Save & Commit**

#### Editing & Comparisons Tips
Due to the nature of 'environment specific' Platform Models, the clone feature in Myst does not yet exist. If your goal is to create a like-for-like model then here are some tips to compare the Myst properties.

1. Click **Platform Model** and select your model
2. Click Actions > **View Report** <br>![[img/model-view-report.png]]
3. Scroll down to **Properties** section
4. Highlight and copy the property(s)/value(s) into your clipboard <br>![[img/model-compare-properties.png]]
5. Paste into a tool such as Microsoft Excel <br>![[img/model-excel.png]]
6. In Excel go to **Save As** and select **CSV** type <br>![[img/model-excel-csv.png]]
7. Repeat for the other environment you want to compare
8. Finally, use a comparison tool to compare the two files <br>![[img/model-compare-final.png]]
9. Fix accordingly.
10. Update all passwords as required, especially if they were environment specific credentials.

**NOTE:** The Platform Blueprint was cloned so in theory most of the changes should be within the Platform Model.

#### Rest API Tip
If you have experience with Rest APIs then you can use that to post/put/get/etc via the [Myst REST API](https://help.mystsoftware.com/?q=rest+api). We have found a tool such as [Postman](https://www.postman.com/) is quite helpful.
