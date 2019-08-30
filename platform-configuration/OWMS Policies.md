### OWSM Policies in MyST:

MyST helps you configure and secure your services in your environment with OWSM Policies. WSM Policies in MyST can be added as per one's requirement, as an Artifact. Applying these policies can be done in two ways.. *Forward* and *Backward*.



### Uploading WSM Policies using Artifactory:

OWSM policies can be built as per one's requirement, and can applied through Artifacts in MyST. Packages(Artifacts) can be added as a 'Third Party Artifact' in MyST from the respected Artifactory. 

**FORWARD Registration to upload WSM Policies as a Third Party Artifact:**

- Upload your policies as a Zip.file using the Artifactory. Once the policies are build as a package, you can use the Artifactory to upload/deploy them.

- Hover onto DEPLOY button and upload the WSM Policy Artifacts

  ![1566294570878](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1566294570878.png)



- Select the Type of Package and drop/upload the files into the Artifactory. 

  ![1566304640584](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1566304640584.png)



### Applying WSM Policies in MyST:

One needs to navigate through Artifacts, to apply the Policies in MyST. Policies need to be registered as a 'Third Party Artifact'.

![1566305990277](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1566305990277.png)



- Fill in the following with the rightful information to register the Artifact.

  ![1566306927551](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1566306927551.png)

  Now you can see that you have applied the Artifacts as OWSM policies, and is being reflected in 'Artifacts' under 'Release Management'.

##### **REVERSE Registration  to apply the WSM Policies:**

- Commit the updated policy files to your source code repository.

- Before that, create the pom.xml file in the specific format as below...

  ![1567143726069](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1567143726069.png)

- <directory> - Mention the location where your code is being put.

- <extrac-files> - Mention the files with a comma(,) if more than one., and with '*', if you want to extract all the files.

- Configure and Build the Jenkins job, to push the WSM package into the Atrifactory.

- Go to MyST Studio and Deploy your WSM Policies in the Pipeline.