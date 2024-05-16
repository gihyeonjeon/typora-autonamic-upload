# why

To avoid the picgo error 422 because the same file is already uploaded in github.

# Goal

Just modify the file name what user want to upload to github with time stamp.

# How

1. download the picgo-core in typora.

2. download the npm.

3. download the picgo using npm.

   ```
   npm install -g picgo
   ```

4. Find the .picgo directory. Usually .picgo directory is in **C:\Users\User\.picgo**.
   Then make a directory whose name starts with "picgo-plugin-".
   In my case, I created the picgo-plugin-test.

   ```
   cd C:\Users\User\.picgo
   mkdir picgo-plugin-test
   cd picgo-plugin-test
   npm init -y
   ```

   Then you can see the new file is created in plugin directory.

5. And make another file named **index.js**

   ```
   type nul > index.js
   ```

   And paste the following code into the file using the familiar editor program.

   ```js
   const fs = require('fs');
   const path = require('path');
   
   module.exports = (ctx) => {
     const register = () => {
       ctx.helper.transformer.register('timestamp', {
         handle: transformerHandle
       });
       ctx.helper.uploader.register('githubTest', {
         handle: uploaderHandle
       });
     };
   
     const transformerHandle = (ctx) => {
       const input = ctx.input;
       const output = input.map(item => {
         const timestamp = new Date().toISOString().replace(/[-:.]/g, '');
         const basename = path.basename(item);
         const newFileName = `${timestamp}-${basename}`;
         return {
           buffer: fs.readFileSync(item),
           fileName: newFileName,
           extname: path.extname(basename)
         };
       });
       ctx.output = output;
       return ctx;
     };
   
     const uploaderHandle = async (ctx) => {
       const output = ctx.output;
       for (let item of output) {
         const url = await uploadToGitHub(item, ctx);
         item.imgUrl = url;
       }
       return ctx;
     };
   
     async function uploadToGitHub(file, ctx) {
       const content = Buffer.from(file.buffer).toString('base64');
       const { fileName } = file;
       const githubPath = `images/${fileName}`;
       const repo = 'yourusername/reponame';
       const branch = 'main';
   
       const body = JSON.stringify({
         message: `upload ${fileName}`,
         content,
         branch
       });
   
       const response = await ctx.request({
         method: 'PUT',
         url: `https://api.github.com/repos/${repo}/contents/${githubPath}`,
         headers: {
           'Authorization': `token ${ctx.config.githubToken}`,
           'Content-Type': 'application/json'
         },
         body
       });
   
       if (response.statusCode === 201) {
         return JSON.parse(response.body).content.download_url;
       } else {
         throw new Error('Failed to upload to GitHub');
       }
     }
   
     return { register, uploader: 'github', transformer: 'timestamp' };
   };
   
   ```

6. And modify the config.json file in the upper directory as follows.
   ```
   cd ..
   ```

   ```json
   {
     "picBed": {
       "current": "github",
       "uploader": "github",
       "github": {
         "repo": "{username}/{repo}",
         "token": "{your_token}",
         "path": "images/",
         "customUrl": "",
         "branch": "main",
         "customUploadName": "{{fileName}}-{{date:YYYYMMDDHHmmss}}"
       },
       "transformer": "timestamp"
     },
     "picgoPlugins": {
       "picgo-plugin-test": true
     }
   }
   ```

   You have to modify {username}, {repo}, {your_token} to suit your situation.

7. Add the plugin into picgo
   ```
   picgo add ./picgo-plugin-test
   ```

8. Select the uploader, transformer, plugins.

   ```
   picgo use
   ```
