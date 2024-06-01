# Webpack Questions

### **What does npx webpack do?**
<details>

npx webpack runs the Webpack bundler using npx, which is a package runner tool that comes with Node.js. This command:

Executes the locally installed version of Webpack without needing a global installation.
Bundles the JavaScript files and other assets according to the configuration defined in webpack.config.js.
Essentially, it compiles and packages your application resources, making them ready for deployment.

</details>

### **What is the difference between npm and npx?**
<details>

The difference between `npm` and `npx` is:

- **npm (Node Package Manager):** A package manager for Node.js that installs packages globally or locally, and manages dependencies for your projects.
- **npx (Node Package eXecute):** A tool that comes with `npm` to execute packages directly from the command line without needing a global installation, allowing you to run Node binaries from `node_modules` or directly from the npm registry.

</details>

### **How can, in webpack, can one dynamically bundle CSS dependencies into a webpack build?**
<details>

Loaders in Webpack are modules that transform the source code of a module before it is bundled. They allow you to preprocess files as you `import` or `load` them.

- **css-loader:** Interprets `@import` and `url()` like `import/require()` and resolves them.
- **style-loader:** Injects CSS into the DOM by creating `<style>` tags.

By using these loaders together, you can import CSS files in JavaScript, and Webpack will bundle the CSS into the final build, dynamically inserting it into the HTML at runtime.
</details>

### **How do Webpack's built-in Asset Modules work to handle image files, and what effects do they have on how images are referenced and processed in JavaScript, CSS, and HTML files?**
<details>

Webpack's built-in Asset Modules simplify handling image files by automatically processing and bundling them. They support four types:

1. **asset/resource:** Emits a separate file and exports the URL.
2. **asset/inline:** Exports the image as a base64-encoded string.
3. **asset/source:** Exports the source code of the asset.
4. **asset:** Automatically chooses between resource and inline, based on file size (default limit is 8kb).

**Effects on referencing and processing:**

- **JavaScript:** You can `import` images, and Webpack will handle their paths.
  ```javascript
  import img from './image.png';
  document.getElementById('img').src = img;
  ```

- **CSS:** Webpack processes `url()` in CSS files, adjusting paths and inlining images if configured.
  ```css
  .background {
    background-image: url('./image.png');
  }
  ```

- **HTML:** You can reference images in HTML, and Webpack will adjust paths or inline them based on the configuration.
  ```html
  <img src="./image.png" />
  ```

Asset Modules ensure images are optimized, correctly referenced, and bundled with the rest of your assets.

</details>

### **How do you setup a DEV testing server using webpack-dev-middleware?**
<details>
To set up a development testing server using `webpack-dev-middleware`, follow these steps:

1. **Install necessary packages:**
   ```sh
   npm install webpack webpack-cli webpack-dev-middleware express --save-dev
   ```

2. **Create a basic Webpack configuration (`webpack.config.js`):**
   ```javascript
   const path = require('path');

   module.exports = {
     entry: './src/index.js',
     output: {
       filename: 'bundle.js',
       path: path.resolve(__dirname, 'dist'),
       publicPath: '/',
     },
     mode: 'development',
   };
   ```

3. **Set up the Express server (`server.js`):**
   ```javascript
   const express = require('express');
   const webpack = require('webpack');
   const webpackDevMiddleware = require('webpack-dev-middleware');
   const webpackConfig = require('./webpack.config.js');

   const app = express();
   const compiler = webpack(webpackConfig);

   app.use(
     webpackDevMiddleware(compiler, {
       publicPath: webpackConfig.output.publicPath,
     })
   );

   app.listen(3000, function () {
     console.log('Dev server listening on port 3000');
   });
   ```

4. **Run the server:**
   ```sh
   node server.js
   ```

This setup will serve your Webpack bundle via Express, enabling hot-reloading and efficient development.
</details>

### **What does webpack-dev-middleware do?**
<details>
webpack-dev-middleware is a middleware for a Node.js server that serves Webpack bundles directly from memory, which speeds up the development process by avoiding disk writes. It intercepts HTTP requests and dynamically serves the latest compiled assets, providing real-time updates when source files change. This middleware is typically used with development servers like Express.js, enabling efficient and seamless integration with Webpack, and supports features like Hot Module Replacement (HMR) for instant module updates without full page reloads.
</details>

### **What are the downside of splitting modules using different entries only?**
<details>
Splitting modules using different entries in Webpack has several downsides:

1. **Increased Complexity:** Managing multiple entry points can complicate the configuration and build process.
2. **Redundant Code:** Common dependencies may be included in multiple bundles, leading to increased bundle sizes and redundancy.
3. **Longer Load Times:** Multiple requests for different bundles can slow down initial page load times.
4. **Caching Issues:** Changes in one entry point can invalidate the cache for other bundles if not managed correctly.

Overall, it can lead to inefficient bundling and resource management.
</details>

### **What strategies can be employed to prevent duplication?**
<details>
To prevent duplication in Webpack, you can employ the following strategies:

1. **CommonChunksPlugin / SplitChunksPlugin:** Use Webpack's built-in optimization plugins to split common dependencies into separate chunks.
   ```javascript
   module.exports = {
     optimization: {
       splitChunks: {
         chunks: 'all',
       },
     },
   };
   ```

2. **Shared Libraries:** Externalize libraries used across multiple entry points by specifying them as external dependencies, or by using Webpack's DLL (Dynamic Link Library) plugin.

3. **Code Splitting:** Dynamically import modules to create separate bundles for code that is only used in specific parts of the application.
   ```javascript
   import(/* webpackChunkName: "my-chunk-name" */ './module.js').then((module) => {
     // Use the module
   });
   ```

4. **Caching:** Configure long-term caching to ensure that unchanged modules are cached by the browser, reducing the need to re-download them.

These strategies help to optimize bundle size and load times by efficiently managing dependencies and reducing redundancy.
</details>

### **What does a config with optimization.splitChunks.cacheGroups.vendor do?**
<details>
A configuration with `optimization.splitChunks.cacheGroups.vendor` in Webpack is used to optimize the bundling process by separating third-party (vendor) libraries from the application code. This helps in reducing redundancy and improving caching. Here's what it does:

1. **Identifies Vendor Code:** It creates a separate chunk for modules coming from `node_modules`, which are typically third-party libraries.

2. **Improves Caching:** Since vendor code changes less frequently than application code, separating it allows for better caching. Users only need to re-download the vendor chunk when dependencies change.

3. **Optimizes Load Time:** Smaller, more focused bundles can be loaded faster and in parallel, improving initial load times.

#### Example Configuration
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
};
```

#### How It Works
- **`test:`** Identifies which modules to include in this chunk (here, any module from `node_modules`).
- **`name:`** Specifies the name of the chunk (here, `vendors`).
- **`chunks:`** Determines which chunks will be selected (here, all chunks).

#### Benefits
- **Reduces duplication:** Shared vendor code is bundled once.
- **Enhances caching:** Separate vendor bundles are cached independently.
- **Speeds up build process:** Clear separation of code improves build efficiency.
</details>

### **What does "cache_groups" indicate?** 
<details>
In Webpack, cacheGroups within the splitChunks configuration specifies how modules should be grouped into separate bundles. It allows for custom grouping based on rules like module location (e.g., node_modules), file types, or other criteria, optimizing the build process by controlling the splitting and caching of chunks.
</details>

### **What is HtmlWebpackPlugin?**
<details>
HtmlWebpackPlugin is a Webpack plugin that simplifies the creation of HTML files to serve your bundled JavaScript. It automatically injects your script tags and supports template customization.
</details>

### **When would one use router.use() vs app.use()**?

<details>

- router.use() is used within an Express Router instance to apply middleware to specific routes or route groups.
- app.use() is used within the main Express application instance to apply middleware globally to all routes.
</details>

### **What are ECMA modules?**
<details>
ECMA modules, also known as ES6 modules, are a standardized module system in JavaScript that allows for the use of import and export statements to include and share code between files, providing a clean and efficient way to manage dependencies.
</details>

### **What are source maps in webpack?**
<details>
Source maps in Webpack are files that map the minified or bundled code back to the original source code. They help developers debug their code by providing a way to see the original source in the browser's developer tools, making it easier to trace errors and understand the code during development.
</details>

### **What are the main source maps used and what differences exist between them?**
<details>
The main types of source maps used in Webpack include:

1. **`eval`**
   - **Performance:** Fastest build speed.
   - **Quality:** Maps generated code to source with inline comments.
   - **Use Case:** Ideal for development with frequent builds.

2. **`source-map`**
   - **Performance:** Slower build speed.
   - **Quality:** High-quality external source maps.
   - **Use Case:** Suitable for production for accurate debugging without exposing source.

3. **`cheap-source-map`**
   - **Performance:** Medium build speed.
   - **Quality:** Only line mappings, no column mappings.
   - **Use Case:** Good for development where speed and minimal detail are sufficient.

4. **`cheap-module-source-map`**
   - **Performance:** Medium build speed.
   - **Quality:** Similar to `cheap-source-map` but includes loader processing.
   - **Use Case:** Useful in development for faster rebuilds.

5. **`inline-source-map`**
   - **Performance:** Slower build speed due to large bundle size.
   - **Quality:** Full source maps inlined within the code.
   - **Use Case:** Useful for small projects or debugging specific issues.

6. **`hidden-source-map`**
   - **Performance:** Similar to `source-map`.
   - **Quality:** External source maps but not referenced in the code.
   - **Use Case:** Suitable for production where you want source maps available but not exposed to end-users.

#### Differences
- **Build Speed:** Varies from very fast (`eval`) to slower (`source-map`).
- **Detail Level:** Varies from basic line mappings (`cheap-source-map`) to full detail with columns and original source (`source-map`).
- **Use Cases:** Some are better for development (`eval`, `inline-source-map`), others for production (`source-map`, `hidden-source-map`).
</details>
