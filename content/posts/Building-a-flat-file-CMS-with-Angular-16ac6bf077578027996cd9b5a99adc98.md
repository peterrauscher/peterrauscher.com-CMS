---
title: "Building a flat-file CMS with Angular"
date: "2024-12-28T15:51:00.000Z"
lastmod: "2024-12-28T16:00:00.000Z"
draft: true
series: []
authors:
  - "Peter"
tags: []
categories: []
NOTION_METADATA:
  object: "page"
  id: "16ac6bf0-7757-8027-996c-d9b5a99adc98"
  created_time: "2024-12-28T15:51:00.000Z"
  last_edited_time: "2024-12-28T16:00:00.000Z"
  created_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  last_edited_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  cover: null
  icon: null
  parent:
    type: "database_id"
    database_id: "167c6bf0-7757-810a-95f9-cfb3b2adb3e1"
  archived: false
  in_trash: false
  properties:
    series:
      id: "B%3C%3FS"
      type: "multi_select"
      multi_select: []
    draft:
      id: "JiWU"
      type: "checkbox"
      checkbox: true
    authors:
      id: "bK%3B%5B"
      type: "people"
      people:
        - object: "user"
          id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
          name: "Peter"
          avatar_url: "https://s3-us-west-2.amazonaws.com/public.notion-static.com/c7114a\
            e2-5f13-4f26-95a6-9feccbcca818/profile-pic_(1).png"
          type: "person"
          person:
            email: "39c43lmy@lumail.xyz"
    custom-front-matter:
      id: "c~kA"
      type: "rich_text"
      rich_text: []
    tags:
      id: "jw%7CC"
      type: "multi_select"
      multi_select: []
    categories:
      id: "nbY%3F"
      type: "multi_select"
      multi_select: []
    Last edited time:
      id: "vbGE"
      type: "last_edited_time"
      last_edited_time: "2024-12-28T16:00:00.000Z"
    summary:
      id: "x%3AlD"
      type: "rich_text"
      rich_text: []
    Name:
      id: "title"
      type: "title"
      title:
        - type: "text"
          text:
            content: "Building a flat-file CMS with Angular"
            link: null
          annotations:
            bold: false
            italic: false
            strikethrough: false
            underline: false
            code: false
            color: "default"
          plain_text: "Building a flat-file CMS with Angular"
          href: null
  url: "https://www.notion.so/Building-a-flat-file-CMS-with-Angular-16ac6bf077578\
    027996cd9b5a99adc98"
  public_url: "https://peterrauscher.notion.site/Building-a-flat-file-CMS-with-An\
    gular-16ac6bf077578027996cd9b5a99adc98"
MANAGED_BY_NOTION_HUGO: true

---


A flat-file CMS allows you to manage content for your website without a database backend. Instead, store your pages as regular text files (or in this case, Markdown files), build, and publish! If you’re reading this right now, you know what the result of this looks like. I spent the last two days building my own flat-file CMS for my site, and now I’d love to walk you through how I did it.


Though I’ll skip most of the stylistic choices (because they’re mostly irrelevant) this framework can be used to build a custom blog on your own Angular website, without the need to individually create new components for each post, manage a database, or write your posts in HTML format. Let’s get started!


## Converting posts from Markdown to HTML


Writing in markdown is super convenient, and supported by just about any text editor. To convert these `.md` files to browser-ready HTML, I wrote a simple little Node.js script using two great npm packages called [gray-matter](https://www.npmjs.com/package/gray-matter) and [showdown](https://www.npmjs.com/package/showdown).


The `gray-matter` package lets us define Jekyll-style front matter for our posts, so we can add metadata for each that we’ll use later in the build script - like so:


```yaml
---
title: Some blog post title
date: 2020-01-24
published: true
thumbnail: some-pic.jpg
permalink: some-post
---
```


The `showdown` package converts the markdown content to HTML. For example, this Markdown:


```markdown
Some blog post introduction...
## A sub-heading
Some section content...
```


Will look like this:


```html
<p>Some blog post introduction...</p>
<h2>A sub-heading</h2>
<p>Some section content...</p>
```


First, the script finds all of the relevant files. It checks the `posts` folder, as defined by the `MARKDOWN_DIRECTORY`variable:


```javascript
const MARKDOWN_DIRECTORY = path.join(__dirname, "posts");
```


Next, it runs the following code, which reads each of the markdown files from the post into a global array of objects called `files`:


```javascript
let files = [];
files = fs
  .readdirSync(MARKDOWN_DIRECTORY)
  .map((f) => path.join(MARKDOWN_DIRECTORY, f))
  .filter((f) => fs.lstatSync(f).isFile())
  .map((file) => {
    // Read the whole file to a string
    let fileContent = fs.readFileSync(file).toString();
    // Parse it with gray-matter
    let post = matter(fileContent);
    post.filename = file;
    // Convert the content to HTML with showdown
    post.content = converter.makeHtml(post.content);
    return post;
  });
```


The line `let post = matter(fileContent)` parses the file content with front matter into an object that looks like:


```javascript
{
    "data": {
        "title":"Some blog post title",        "date":"2020-01-24T00:00:00.000Z",        "published":true,        "thumbnail":"some-pic.jpg",        "permalink":"some-post"    },    "content":"Some blog post introduction...\n## A sub-heading\nSome section content...",}
```


And the line `post.content = converter.makeHtml(post.content)` converts the `content` field to HTML, as you’d expect:


```javascript
{
  "data": {
    "title": "Some blog post title",
    "date": "2020-01-24T00:00:00.000Z",
    "published": true,
    "thumbnail": "some-pic.jpg",
    "permalink": "some-post"
  },
  "content": "Some blog post introduction...\n## A sub-heading\nSome section content...",
}
```


Since I wanted to have thumbnails for each post (the file it uses is defined by the `thumbnail` field in the front matter), I decided to add some code that copies the files from our `posts` folder to the `assets` folder in Angular. This way, we can easily reference them in our Angular code. However, some of the image files I would download from [Unsplash.com](https://unsplash.com/) (fantastic public domain images btw) were huge and would take a few seconds to load in the browser. So, I delegated this task to `gulp`, where the files would first be piped to `imagemin` and then copied to the `assets` folder. This optimized the images for faster load times. My `gulpfile.mjs` looks like this:


```javascript
const THUMBNAIL_DIRECTORY = "posts/thumbnails";
const THUMBNAIL_OUTPUT_DIRECTORY = "src/assets/thumbnails";
export default () =>
  gulp.src(`${THUMBNAIL_DIRECTORY}/*`)
    .pipe(imagemin())
    .pipe(gulp.dest(THUMBNAIL_OUTPUT_DIRECTORY));
```


## Using the posts in Angular


Great! So our `files` array holds everything we need to write the HTML files, and our thumbnails are in place to be served by our website. Now, we just have to make this available to Angular in some way. Here are a couple of options I considered for achieving this:

- Serve the posts over an API endpoint
- Dynamically generate new components for each post

Ultimately, I decided against these two options because deploying a whole server for this purpose seemed overkill, and dynamically generating components (although cool) would be a fairly big task and go against Angular best practices.


Instead, I decided to leverage one of the coolest things about Angular: it builds everything into a static site and some plain vanilla JS, entirely in the build process. Cool! So let’s have one generic `post` Component, and dynamically load the content into it using client-side JS. So, our build script writes our `files` array to a `.json` file called `posts.json` in the `src` folder of our Angular app. It also writes each post to its own HTML file, but this is just to allow me to easily inspect if the Markdown-to-HTML conversion went off without a hitch on my end:


```javascript
// Write only the published: true posts to posts.json for Angular to serve
fs.writeFileSync(
  OUTPUT_FILE,
  JSON.stringify(files.filter((p) => p.data.published))
);
// For each file
files.forEach((file) => {
  if (file) {
    // Change the extension to .html
    let filename = path.join(
      OUTPUT_DIRECTORY,
      `${path.basename(file.filename, ".md")}.html`
    );
    // Write the file to the build directory
    fs.writeFileSync(filename, file.content);
  }
});
```


Now, I generated two components: `blog` for the page displaying a list of all posts, and `post` for a template post page that can be loaded with whatever content.


### But first, Routing


We’ll keep this part simple. In our `app-routing.module.ts` file, we have the following routes:


```typescript
const routes: Routes = [
    ...    { path: 'blog', component: BlogComponent },    { path: 'blog/:postlink', component: PostComponent },    { path: 'not-found', component: PageNotFoundComponent },    { path: '**', redirectTo: '/not-found' },    ...]
```


### The Post component


So, our `post.component.html` component is the generic page where all posts will have their content loaded. Here, the classes are from the [Bulma CSS framework](https://bulma.io/), and the template looks like this:


```html
<div class="main section container pt-0">  <div class="block is-fullwidth">    <h1 class="title is-1">{{ meta.title }}</h1>    <p class="has-text-grey">Published {{ meta.date }}</p>  </div>  <div class="content" [innerHTML]="content"></div></div>
```


Note the inline directive, `[innerHTML]="content"` on the content div. This takes the component’s `content` property and loads the raw HTML into it, creating the actual content of our post. The `{{ meta.title }}` and `{{ meta.date }}` properties define, the title and publishing date of the post (I know, shocking 😱).


And here’s the cool part: the TypeScript code for the component:


```typescript
// A library that provides syntax highlighting for <code> blocksimport hljs from "highlight.js";// The posts.json file we generated in the build stepimport posts from "src/posts.json";interface Post {
  title: string;  date: string;  content: string;  thumbnail: string;  permalink: string;}
// Iterate through each post object, filtering null/undefined itemslet postData = posts.filter(Boolean).map((p) => {
  const post: Post = {
    title: p.data.title,    // Format the date to something readable    date: new Date(p.data.date).toLocaleDateString("en-US", {
      month: "long",      day: "numeric",      year: "numeric",    }),    content: p.content,    // Create the actual link to the thumbnail as it's available on the server    thumbnail: `/assets/thumbnails/${p.data.thumbnail}`,    permalink: p.data.permalink,  };  return post;});
```


Now, we implement the `OnInit` interface so that when the component is initialized, we find the post with a `permalink` attribute matching the current route of the client. If no such post exists, we redirect to the site’s 404 page:


```typescript
ngOnInit(): void {
    this.postlink = this.route.snapshot.params['postlink'];    let postMeta = postData.find((p) => p.permalink === this.postlink);    if (postMeta) {
        this.meta = postMeta;        this.content = postMeta.content;    } else {
        this.router.navigate(['/not-found']);    }
}
```


We also implement the `AfterViewInit` interface, so that we can highlight our code blocks only after all of the content is done loading:


```typescript
ngAfterViewInit(): void {
    this.document.querySelectorAll('code').forEach((el) => {
        hljs.highlightElement(el as HTMLElement);    });}
```


### The Blog component


The final piece of this puzzle is the `Blog` component, which lists all of the posts available on our site. Again, the CSS classes for this template come from Bulma:


```html
<div class="main section container pt-0">  <!-- For each post, create an entry -->  <div *ngFor="let post of posts" class="box is-rounded p-0">    <!-- Link to the post -->    <a class="nostyle" [href]="'/blog/' + post.permalink">      <div class="columns">        <!-- Inline style directive to load the thumbnail as the background image for this div -->        <div class="column is-one-fifth is-thumbnail is-hidden-mobile" [style]="post.thumbnail"></div>        <div class="column is-four-fifths">          <div class="content px-3 py-1">            <!-- Display the post title -->            <h3 class="title is-3 mb-2 no-border">{{ post.title }}</h3>            <!-- Display the first 180 characters of the post as a preview -->            <p class="mb-2" [innerHTML]="post.content.substring(0, 180) + '...'"></p>            <!-- Display the published date -->            <span class="tag is-info is-light">{{ post.date }}</span>          </div>        </div>      </div>    </a>  </div></div>
```


Our blog component pulls the `posts.json` data in much the same way as our post component does, but it makes the whole list available as a `posts` property, which is used by the `*ngFor` directive above:


```typescript
let postData = posts.map((p) => {
  const post: Post = {
    title: p.data.title,    date: new Date(p.data.date).toLocaleDateString("en-US", {
      month: "long",      day: "numeric",      year: "numeric",    }),    content: p.content,    thumbnail: `background-image: url("/assets/thumbnails/${p.data.thumbnail}"); background-position: center; background-size: cover;`,    permalink: p.data.permalink,  };  return post;});export class BlogComponent {
  posts = postData;}
```


## Deployment


The final step is obviously to deploy the site, which I do with AWS. First, when I push to Github, any new posts trigger an AWS CodeBuild run which builds the site and copies the generated files to an S3 bucket. Luckily, the site is all static and can be served just over S3 with no need for a web server. I also have a CloudFront distribution set up for the bucket, so files can be cached in edge locations and available worldwide with low latency. My `buildspec.yml` looks like this:


```yaml
version: 0.2env:  variables:    BUCKET_NAME: peterrauscher.com    DISTRIBUTION_ID: E34HJHH2D2T6HSphases:  pre_build:    commands:      - echo Installing source NPM dependencies...      - npm install      - npm install -g @angular/cli      - npm install -g gulp-cli  build:    commands:      - echo Build started on `date`      - echo Compiling the dist folder...      - npm run build  post_build:    commands:      - echo Build completed on `date`      - echo Deleting existing site files...      - aws s3 sync dist s3://${BUCKET_NAME}/ --delete      - echo Invalidating CloudFront cache...      - aws cloudfront create-invalidation --distribution-id ${DISTRIBUTION_ID} --paths "/*"artifacts:  files:    - "**/*"  discard-paths: no  base-directory: "dist/peterrauscher.com"
```


## Wrapping up


And that’s about it! We’ve covered all of the logic needed to set up a flat-file blog on your own Angular site. Feel free to tweak things like the routing/paths, stylistic choices, or removing features you don’t intend to use, but I’m quite happy with how this worked out and happy that I was able to implement it statically so I could still run it purely on S3. I hope you enjoyed the read and learned something you can apply on your own!

