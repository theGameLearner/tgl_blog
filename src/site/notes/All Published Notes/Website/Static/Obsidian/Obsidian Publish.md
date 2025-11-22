---
{"dg-publish":true,"permalink":"/all-published-notes/website/static/obsidian/obsidian-publish/"}
---

created: 2025-11-22
updated: 2025-11-22


## Good publishing Options
- To publish an Obsidian vault, we can use [Quartz](https://github.com/jackyzha0/quartz) but the pop up for all web-links is kind of irritating if we want to publish it for all.
	- Guide [link](https://quartz.jzhao.xyz/)
	- how to detailed : [link](https://notes.nicolevanderhoeven.com/How+to+publish+Obsidian+notes+with+Quartz+on+GitHub+Pages)
- [Digital Garden](https://github.com/oleeskild/Obsidian-digital-garden) - oleeskild 
	- tutorial : [link](https://dg-docs.ole.dev/getting-started/01-getting-started/) 
	- Needs a Vercel account.
- [Digital garden](https://github.com/TuanManhCao/digital-garden) - TuanManhCao
	- lot of Digital Garden tutorials and options are popular. This also covers using Netlify for publishing.
- Next.js and Vercel
	- how to : [reddit](https://www.reddit.com/r/ObsidianMD/comments/zz89p1/how_to_publish_obsidian_notes_for_free_nextjs/)
	- [YouTube](https://www.youtube.com/watch?v=rKSpK1dXn4E) tutorial using CI/CD.
	- https://vercel.com/pricing
- MkDocs
	- [GitHub](https://github.com/jobindjohn/obsidian-publish-mkdocs) 

## My Choice
**My choice is [Digital Garden](https://github.com/oleeskild/Obsidian-digital-garden) from 'oleeskild'**

### How to create a web site repository
Steps Done:
1. Make a new publish Vault to publish documents.
2. We will use this as a test for publishing.
3. Add the plugin [Digital Garden](obsidian://show-plugin?id=digitalgarden).
4. create a [Vercel](https://vercel.com/signup) Account using GitHub, giving limited access to the repos.
	1. Scope: your GitHub organisation name / User name
	2. name: name of repository which will be published.
	3. This is adding Vercel as an app in GitHub
5. open the GitHub on a browser, create a new [Personal Access Token](https://github.com/settings/personal-access-tokens/new):
	1. give repository access for the new repository created by Vercel.
	2. in permissions: Repository permissions :
		1. contents - read and write
		2. Pull requests - read and write
	3. Confirm / Generate token
		1. this is generated Token with no expiry
6. Open Obsidian, in plugins settings for Digital garden, add the repo name, user name and the token generated.
7. Now, there are a few properties that define the home directory and the notes that are published.
	1. `dg-publish` : This property tells Vercel who has read and write access that the note needs to be published
	2. `dg-home`: This is added '1' time to a note which acts as the home page for our repository.
8. open obsidian's command palette, select "==Digital Garden: Publish Single note==" to push the note you have finished editing.
9. If the note is published, you can directly go to your webpage [URL](https://tglblog.vercel.app/) to see the change.
10. If you want to un-publish a note, without deleting the note from your vault, simply un-check or remove the `dg-publish` property in the note, open the [publication center](https://dg-docs.ole.dev/getting-started/02-commands/#open-publication-center) and click the "Delete notes from garden" button.
11. Further, used the reference from [this](https://k9bdm.com/digital-garden-setup/3-add-site-logo/) link or [this](https://bradleydonmorris.com/digital-garden-setup/3-add-site-logo/) link to add logo icon for the site.
	1. Open the project's github page, find file: `/src/site/styles/custom-style.scss`
	2. this file is used to override style settings, so you can add new logo here.
	3. For Logo in browser's tab as well as for a logo above the sidebar:

`src/site/styles/custom-style.scss`:

```scss title:src/site/styles/custom-style
body {
    /***
      ADD YOUR CUSTOM STYLING HERE. (INSIDE THE body {...} section.)
      IT WILL TAKE PRECEDENCE OVER THE STYLING IN THE STYLE.CSS FILE.
   ***/
    //  background-color: white;
    //  .content {
    //   font-size: 14px;
    //  }
    //  h1 {
    //   color: black;
    //  }
}

// Add a logo on top left of the screen above filetree sidebar
nav.filetree-sidebar {
  top: 3em; // padding from the top of the page
  padding-top: 7em; // content of this div will leave space to show the logo above the top bar
  height: calc(100% - 7em); // Adjust height of the filetree-sidebar accordingly
  h1::before { 
    content: " ";
    display: block;
    width: 100%;
    height: 3em; // This is the height of your logo image
    position: absolute;
    top: 0;
    left: 0;
    background-size: contain;
    background-repeat: no-repeat;
    background-image: url("/img/user/Images/Logo/Tgl logo.png");  // URL of the image being used in the sidebar
    background-position: left center; // Horizontal left, vertical center
  }
}

// Add a logo in the browser's tab for this page
nav.navbar {
  .navbar-inner {
    h1::before {
      content: " ";
      display: inline-block;
      width: 1em;
      height: 1.2em;
      background-size: contain;
      background-repeat: no-repeat;
      background-image: url("/img/user/Images/Logo/Tgl logo.png");
      margin-right: 5px;
      background-position: bottom;
    }
  }
}

```

The image is located in "/Images/Logo/Tgl logo.png" path in the vault.
everything after "/img/user/" is your obsidian vault files.

> [!Note]
> If the image is not used in any of your documents, it does not get published, so there is a chance that it will not show up.

### If deployment fails
Deployment needs to be checked in Vercel, the site data is checked in github.
I will try to update any issues I face durin deployment.
#### Eleventy updated the code
Vercel uses 'Eleventy' and if this code is updated, we may face the issue where Vercel may not be able to deploy the code.
Code block in vercel:
```sh
09:02:18.947 [11ty] Unfortunately you’re using code that monkey patched some Eleventy internals and it isn’t async-friendly. Change your code to use the async `read()` method on the template instead!
```
**Fix**:
- Open the Obsidian vault -> Settings
- Community Plugins -> Digital Garden -> Update Site -> **"Create PR"**
	- This tells your git hub repository that you want to create the pull request for your Digital Garden to be updated with the latest code. 
- Open github repository to look at the pull request. (Obsidian shows the [link](https://github.com/theGameLearner/tgl_blog/pull/21) of the pull request as well one time, you can click it and open it)
- merge the pull request
- Open Obsidian and make another push to single or multiple files.
- or you can open [vercel](https://vercel.com/rishabh-jains-projects-78a8e192/tgl_blog/deployments) and deploy from there.



---

[^1]: The sire logo does not appear
[^2]:

