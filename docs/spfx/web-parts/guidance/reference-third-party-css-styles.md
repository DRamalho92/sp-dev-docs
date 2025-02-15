---
title: Reference third-party CSS styles in SharePoint Framework web parts
description: Two approaches to including third-party CSS styles in web parts, and how each approach affects the resulting web part bundle.
ms.date: 12/02/2020
ms.prod: sharepoint
ms.localizationpriority: high
---

# Reference third-party CSS styles in SharePoint Framework web parts

To build rich SharePoint Framework client-side web parts, you can leverage many third-party libraries. Also to scripts, these libraries often contain additional assets such as stylesheets. This article describes two approaches to including third-party CSS styles in web parts, and how each approach affects the resulting web part bundle. The example web part discussed in this article uses jQuery and jQuery UI to display an accordion.

![jQuery UI accordion rendered by a SharePoint Framework client-side web part](../../../images/thirdpartycss-accordion-styled.png)

> [!NOTE]
> Before following the steps in this article, be sure to [set up your SharePoint client-side web part development environment](../../set-up-your-development-environment.md).

## Prepare the project

### Create a new project

1. Create a new folder for your project.

    ```console
    md js-thirdpartycss
    ```

1. Go to the project folder.

    ```console
    cd js-thirdpartycss
    ```

1. In the project folder, run the SharePoint Framework Yeoman generator to scaffold a new SharePoint Framework project.

    ```console
    yo @microsoft/sharepoint
    ```

1. When prompted, enter the following values:

    - **js-thirdpartycss** as your solution name
    - **Use the current folder** for the location to place the files
    - **No JavaScript framework** as the starting point to build the web part
    - **jQuery accordion** as your web part name
    - **Shows jQuery accordion** as your web part description

    ![The SharePoint Framework Yeoman generator with the default choices](../../../images/thirdpartycss-yeoman.png)

1. Open your project folder in your code editor. This article uses Visual Studio Code in the steps and screenshots, but you can use any editor that you prefer.

    ![The SharePoint Framework project open in Visual Studio Code](../../../images/thirdpartycss-visual-studio-code.png)

### Add test content

In the code editor, open the **./src/webparts/jQueryAccordion/JQueryAccordionWebPart.ts** file, and then change the `render()` method to:

```typescript
export default class JQueryAccordionWebPart extends BaseClientSideWebPart<IJQueryAccordionWebPartProps> {
  // ...
  public render(): void {
    this.domElement.innerHTML = `
      <div>
        <div class="accordion">
          <h3>Information</h3>
          <div>
            <p>
            The Volcanoes, crags, and caves park is a scenic destination for
            many visitors each year. To ensure everyone has a good
            experience and to preserve the natural beauty, access is
            restricted based on a permit system.
            </p>
            <p>
            Activities include viewing active volcanoes, skiing on mountains,
            walking across lava fields, and caving (spelunking) in caves
            left behind by the lava.
            </p>
          </div>
          <h3>Snow permit</h3>
          <div>
            <p>
            The Northern region has snow in the mountains during winter.
            Purchase a snow permit for access to approved ski areas.
            </p>
          </div>
          <h3>Hiking permit</h3>
          <div>
            <p>
            The entire region has hiking trails for your enjoyment.
            Purchase a hiking permit for access to approved trails.
            </p>
          </div>
          <h3>Volcano access</h3>
          <div>
            <p>
            The volcanic region is beautiful but also dangerous. Each
            area may have restrictions based on wind and volcanic
            conditions. There are three type of permits based on activity.
            </p>
            <ul>
              <li>Volcano drive car pass</li>
              <li>Lava field access permit</li>
              <li>Caving permit</li>
            </ul>
          </div>
        </div>
      </div>`;

      ($('.accordion', this.domElement) as any).accordion();
  }
  // ...
}
```

If you build the project now, you get an error stating that `$` is undefined. This is because the project refers to jQuery without loading it first. There are two approaches to loading the libraries. Neither approach impacts how you use the scripts in code.

## Approach 1: Include third-party libraries in the bundle

The easiest way to reference a third-party library in SharePoint Framework projects is to include it in the generated bundle. The library is installed as a package and referenced in the project. When bundling the project, Webpack picks up the reference to the library and includes it in the generated bundle.

### Install libraries

1. Install jQuery and jQuery UI by running the following command:

    ```console
    npm install jquery jquery-ui --save
    ```

1. Because you're building your web part in TypeScript, you also need TypeScript type declarations for jQuery that you can install by running the following command:

    ```console
    npm install @types/jquery --save-dev
    ```

### Reference libraries in the web part

After installing libraries, the next step is to reference them in the project.

1. In the code editor, open the **./src/webparts/jQueryAccordion/JQueryAccordionWebPart.ts** file. In its top section, just under the last `import` statement, add references to jQuery and jQuery UI.

    ```typescript
    import * as $ from 'jquery';
    require('../../../node_modules/jquery-ui/ui/widgets/accordion');
    ```

    Because you installed the TypeScript type declarations for the jQuery package, you can reference it by using an `import` statement. However, the jQuery UI package is built differently. Unlike how many modules are structured, there's no main entry point with a reference to all components that you can use. Instead, you refer directly to the specific component that you want to use. The entry point of that component contains all references to dependencies that it needs to work correctly.

1. Confirm that the project is building by running the following command:

    ```console
    gulp serve
    ```

1. After adding the web part to the canvas, you should see the accordion working.

    ![jQuery UI accordion without styles rendered by a SharePoint Framework client-side web part](../../../images/thirdpartycss-accordion-not-styled.png)

At this point, you've referenced only the jQuery UI scripts, which explain why the accordion isn't styled. Next, you add the missing CSS stylesheets to brand the accordion.

### Reference third-party CSS stylesheets in the web part

Adding references to third-party CSS stylesheets that are a part of the packages installed in the project is as simple as adding references to the packages themselves. The SharePoint Framework offers standard support for loading CSS files through Webpack.

1. In the code editor, open the **./src/webparts/jQueryAccordion/JQueryAccordionWebPart.ts** file. Just under the last `require` statement, add references to jQuery UI accordion CSS files.

    ```typescript
    require('../../../node_modules/jquery-ui/themes/base/core.css');
    require('../../../node_modules/jquery-ui/themes/base/accordion.css');
    require('../../../node_modules/jquery-ui/themes/base/theme.css');
    ```

    Referencing CSS files that are part of a package in the project is similar to adding references to JavaScript files. All you need to do is specify the relative path to the CSS file that you want to load, including the **.css** extension. When bundling the project, Webpack processes these references and includes the files in the generated web part bundle.

1. Confirm that the project is building by running the following command:

    ```console
    gulp serve
    ```

The accordion should be displayed correctly and branded by using the standard jQuery UI theme.

![jQuery UI accordion branded using the default jQuery UI theme rendered by a SharePoint Framework client-side web part](../../../images/thirdpartycss-accordion-styled.png)

### Analyze the contents of the generated web part bundle

The easiest way to use third-party libraries and their resources is by including them in the generated web part bundle. In this approach Webpack automatically resolves all dependencies between the different libraries and ensures that all scripts are loaded in the correct order. The downside of this approach is that all referenced resources are loaded separately with every web part. So if you've multiple web parts in your project, all using jQuery UI, each web part loads its own copy of jQuery UI and slows down the page.

To see the impact of the libraries on the size of the generated web part bundle, after bundling the project, open the **./temp/stats/js-thirdpartycss.stats.html** file in the web browser. Move your mouse over the chart and you'll see, for example, that the jQuery UI CSS files referenced by the web part make up over 6% of the total web part bundle size.

![jQuery UI CSS highlighted in the chart illustrating the size of the different pieces of the generated web part bundle](../../../images/thirdpartycss-jquery-ui-css-size.png)

As mentioned in the disclaimer following the chart, the sizes are indicative and reflect the size of the debug version of the bundle. The release version of the bundle is smaller. Still, it's good to realize which different pieces compose the web part bundle and their relative size compared to other elements in the bundle.

## Approach 2: Load third-party libraries from a URL

Another way to reference third-party libraries in the SharePoint Framework is by referencing them from a URL such as a public CDN or a privately managed location. The biggest benefits that if you're loading a frequently used library from a public location, there's a chance that users might already have that particular library downloaded on their computer. In that case, the SharePoint Framework reuses the cached library, loading your web part faster.

Even if you can't use a public CDN to load libraries from a central location, its a good practice from the performance point of view. Pointing to a URL allows your users to download the script only once and reuse it across the whole portal, significantly speeding up loading pages and improving the user experience.

When loading third-party libraries from public URLs, keep in mind that there's a risk involved in using those libraries. Because you don't manage the hosting location of any particular script, you can't be sure of its contents. Scripts loaded by the SharePoint Framework run under the context of the current user and are allowed to do whatever that user can do. Also, if the hosting location is offline, your web part won't work.

### Install type declarations for libraries

When you reference third-party libraries from a URL, you don't need to install them as packages in your project. You do have to install their TypeScript type declarations if you want the benefit of type safety checks during development.

Assuming you start with an empty project created as described previously in this article, install TypeScript type declarations for jQuery by running the following command:

```console
npm install @types/jquery --save-dev
```

### Specify URLs of libraries

To load third-party libraries from a URL, you've to specify the URL where they're located in the configuration of your project. In the code editor, open the **./config/config.json** file. In the `externals` section, add the following JSON:

```json
{
  //...
  "externals": {
    //...
    "jquery": "https://code.jquery.com/jquery-3.1.1.min.js",
    "jquery-ui": "https://code.jquery.com/ui/1.12.1/jquery-ui.min.js"
    //...
  }
  //...
}
```

### Reference libraries from the URL in the web part

Having specified the URL that the SharePoint Framework should use to load jQuery and jQuery UI, the next step is to reference them in the project.

1. In the code editor, open the **./src/webparts/jQueryAccordion/JQueryAccordionWebPart.ts** file. In its top section, just under the last `import` statement, add the following references to jQuery and jQuery UI:

    ```typescript
    import * as $ from 'jquery';
    require('jquery-ui');
    ```

    Compared to how you were referencing both libraries when they were installed as packages in your project, referencing them from the URL is similar. Because jQuery has its TypeScript type declarations installed, it can be referenced by using an `import` statement. For jQuery UI, all you want is to load the script on the page.

    Because you registered **jquery** and **jquery-ui** in the project configuration as external resources, when you reference any of these libraries, the SharePoint Framework uses the specified URLs to load them at runtime. When bundling the project, these resources are marked as externals and are excluded from the bundle.

    One difference to keep in mind is that previously you specified to load the accordion from the jQuery UI package, but now you're referring to jQuery UI from the CDN that contains all jQuery UI components.

1. Confirm that the project is building by running the following command:

    ```console
    gulp serve
    ```

    After adding the web part to the canvas, you should see the accordion working.

    ![jQuery UI accordion without styles rendered by a SharePoint Framework client-side web part](../../../images/thirdpartycss-accordion-not-styled.png)

1. In your web browser, open the developer tools, switch to the tab showing the network requests, and reload the page. You should see how both jQuery and jQuery UI are loaded from the CDN.

    ![jQuery and jQuery UI request highlighted in Microsoft Edge developer tools](../../../images/thirdpartycss-libraries-cdn.png)

At this point you've referenced only the jQuery UI scripts, which explain why the accordion isn't styled. Next, you add the missing CSS stylesheets to brand the accordion.

### Reference third-party CSS stylesheets from URL in the web part

Adding references to third-party CSS stylesheets from a URL is different from referencing resources from project packages. While the project configuration in the **config.json** file allows you to specify external resources, it applies only to scripts. To reference CSS stylesheets from a URL, you must use the `SPComponentLoader` instead.

#### Load CSS from the URL using the SPComponentLoader

1. In the code editor, open the **./src/webparts/jQueryAccordion/JQueryAccordionWebPart.ts** file. In the top section of the file, just after the last `import` statement, add the following code:

    ```typescript
    import { SPComponentLoader } from '@microsoft/sp-loader';
    ```

1. In the same file, override the `onInit()` method as follows:

    ```typescript
    export default class JQueryAccordionWebPart extends BaseClientSideWebPart<IJQueryAccordionWebPartProps> {
      protected onInit(): Promise<void> {
        SPComponentLoader.loadCss('https://code.jquery.com/ui/1.12.1/themes/base/jquery-ui.min.css');
        return super.onInit();
      }

      // ...
    }
    ```

    When the web part is instantiated on the page, it loads the jQuery UI CSS from the specified URL. This CSS stylesheet is the combined and optimized version of the jQuery UI CSS that contains the basic styles, theme, and styling for all components.

1. Confirm that the project is building by running the following command:

    ```console
    gulp serve
    ```

The accordion should be displayed correctly and branded by using the standard jQuery UI Theme.

![jQuery UI accordion branded using the default jQuery UI theme rendered by a SharePoint Framework client-side web part](../../../images/thirdpartycss-accordion-styled.png)

### Analyze the contents of the generated web part bundle loading resources from URL

After building the project in the web browser, open the **./temp/stats/js-thirdpartycss.stats.html** file.

Notice how the overall bundle is smaller (7 KB compared to over 300 KB when including jQuery and jQuery UI in the bundle), and how jQuery and jQuery UI are not listed in the chart because they're loaded at runtime.

## See also

- [SharePoint Framework Overview](../../sharepoint-framework-overview.md)
