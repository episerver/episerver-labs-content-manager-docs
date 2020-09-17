> For Content Manager docs, please visit [Content Manager](Readme.md)

# Episerver Grid View

## Install

```Install-Package EPiServer.Labs.GridView```

Link to [nuget](https://nuget.episerver.com/package/?id=EPiServer.Labs.GridView) package.

GridView package is an add-on that has several features:
* [GridView UI search component](#gridview-ui-search-component) - a powerful dgrid-based, fully configurable UI component that flattens the hierarchical structure and displays descendents of the currently selected content item.
* [GridView Main Navigation Component](#gridview-main-navigation-component) - instead of displaying the site in a hierarchical way the editor will be able to browse it as folders (a bit like Windows Explorer).
* [Enhanced Properties](#enhanced-properties) - enhanced ContentReference, ContentArea and LinkItemCollection properties - much more powerful way of finding the needed content items

## GridView UI search component

To enable Grid View for all content types, you can set `IsViewEnabled` of
[GridViewOptions](#gridviewoptions) to true (by default it's false).

But recommended way is to enable custom view only for specific page types. In order to add the UI search component you will just have to register it as a `SearchContentView` and provide a generic type parameter that defines which content types this new View will be applied to.
For example to add the Grid View to all instances of `StandardPage` you will have to add this code to your site:
```cs
[ServiceConfiguration(typeof(ViewConfiguration))]
public class ContainerGridView : SearchContentView<StandardPage>
{
}
```

![Grid View](images/grid-view/grid-view.png "Grid View")

### Configuring columns

It is possible to define which columns are shown in the grid.
For example, editors with multilanguage sites would like to manage languages from the grid. Others may want to see properties related with content publication like “Published By” and “Published On”. Also would be good to display properties specific to for the site like “Hero image” or “Short description”. 

You can show, both standard properties like “Name”, “Status”, “Include in menu” and any custom property added to the model.

Columns are configured on the server side. To simplify configuration 
code there is a builder class. 

The sample columns configuration for StandardPage looks like this:

```cs
[UIDescriptorRegistration]
public class StandardPageGridViewUIEditorDescriptor : ExtendedUIDescriptor<StandardPage>
{
    public SitePageDataUIEditorDescriptor()
        : base(ContentTypeCssClassNames.Page)
    {
        GridSettings = new GridSettings
        {
            Columns = new ColumnsListBuilder()
                .WithContentName()
                .WithContentStatus()
                .WithContentTypeName()
                .WithVisibleInMenu()
                .WithCreatedBy()
                .WithPublishDate()
                .WithCurrentLanguageBranch()
                .WithEdit()
                .WithPreviewUrl()
                .WithActionMenu()
                .Build()
        };
    }
}
````

### Column renderers

The column is not just property value rendered as a text. There are set 
of cell renderers for displaying images, content references or dates. 
For example the image renderer displays thumbnail. After editor clicks on 
the thumbnail the popover with higher resolution and link to the image are displayed.

![Renderes](images/grid-view/grid-view-renderers.gif "Renderes")

The list of built-in renderers is quite long. It inclides renderers like:

* Name – with name of the content, content icon and language information (same as in navigation tree)
* Status – localized content status
* Type – localized content type name
* Languages – displays list of all site languages
* Dates – formatted date
* Content Link – with editable and previewable versions

### Custom renderers

Developer can implement custom renderer. For example when ProductPage type 
has MajorVersion, MinorVersion, Relase Version, BuildVersion integer 
properties and we want to include this information on the grid.

![Custom renderes](images/grid-view/grid-view-custom-renderes.png "Custom renderes")

We probably would like to show it as one column with dot separated values. In this example it wil be “10.7.1.234”.
With custom renderer it will be display like:

![Custom renderes list](images/grid-view/grid-view-custom-renderes-list.png "Custom renderes list")

Here you can find example about [how to create custom column renderer](GridView_CustomRenderer.md).

## Grid menu column

Grid View has context menu available next to current content name and in 
every row of children. Commands provider is shared between the View and main 
navigation. It means that view will display same actions as the tree. 
Even when you implement custom tree context menu action by the 
"[epi-cms/plugin-area/navigation-tree](https://world.episerver.com/documentation/developer-guides/CMS/user-interface/plug-in-areas/)"
 it will be showed in the view. 
For example you can add “Preview” command that opens selected page in View Mode.

![Custom commands](images/grid-view/grid-view-custom-commands.png "Custom commands")

## Working with languages

Working with languages in Grid View is similar to Navigation Tree. 
All translated versions and master versions for non-translated pages are 
displayed by default. When translation is missing, content name has italic font style.

![Languages](images/grid-view/grid-view-languages.png "Languages")

To show only translated pages you can uncheck “All languages” filter. 
It’s the same filter as “Show content in current language only” on the Navigation Tree.

![All languages](images/grid-view/grid-view-languages-all-languages.png "All languages")

Grid view has additional feature. There is a special column renderers for managing languages. For every cell it displays all languages available on the site. Missing translations are grayed out. Clicking on the flag will change the language context and page context.

## Configuring view for page type

For ProductsListPage container, which can contains only ProductPage types, 
we would like to show different columns, that for other pages in the system. 
It doable using Grid View, because columns are configured through the extended 
UIDescriptor (ExtendedUIDescriptor). For example there are two UIDescriptors, 
one for SitePageData (the base class for all pages in the solution) and one 
for ProductsListPages container type.

![Columns configuration](images/grid-view/grid-view-columns-configuration.png "Columns configuration")

For SitePageData we show general information:

![Columns site page](images/grid-view/grid-view-columns-site-page-data.png "Columns site page")

But for Products we show thumbnail, version and and categories:

![Columns product page](images/grid-view/grid-view-columns-products.png "Columns product page")

#### Set Grid View as a default view

To set Grid View as a default view for a page type, you can use `UIDescriptor`.
For example, to set Grid View as a default view for folder:

```cs
[UIDescriptorRegistration]
public class ContentFolderUIEditorDescriptor : ContentFolderUIDescriptor
{
    public ContentFolderUIEditorDescriptor()
    {
        DefaultView = SearchContentView.ViewKey;
    }
}
```

As you can see there is a constant for GridView folder name: `SearchContentView.ViewKey`.

## Navigation tree with locked nodes

For nodes with large number of children expanding child nodes can be turned off. 
Those nodes will be showed with a different icon.

![Locked nodes](images/grid-view/grid-view-locked-node.png "Locked nodes")

It can be done through web.config where I added ContentContainers setting. 
Setting stores coma separated list of ContentReferences that should be displayed as 
Container pages and have no expand button.

![web.config nodes](images/grid-view/grid-view-web.config-ContentContainers.png "web.config nodes")

The Administrators can turn on and off nodes directly through the Edit Mode 
by “Manage Containers” button manage locked nodes. Of course without 
ContentReferences added through web.config which are disabled globally. 
For them it’s not possible to turn off containers through the UI.

![locked items 2](images/grid-view/grid-view-locked-items-2.png "locked items 2")

For other pages, command is enabled.

![locked items](images/grid-view/grid-view-locked-items.png "locked items")

It’s a toggle button. When page is not a container, after clicking the button, 
it will become a container. When page was a container, after clicking the button, 
page won’t be a container anymore.

![using lock nodes](images/grid-view/grid-view-converting-page-to-container.gif "using lock nodes")

## GridView Main Navigation Component

Custom view give the overview of the children, but it could be useful to 
locate the content and using D&D add it to the ContentArea. For this reason 
Grid View can be registered in the navigation pane.

![navigation component](images/grid-view/grid-view-Component.png "navigation component")

Columns are also configurable, so you can extend the pane to have more information about the content.

![navigation component configuring columns](images/grid-view/grid-view-main-navigation-component-with-columns.png "navigation component configuring columns")

You can use search to filter the list and D&D items to ContentReference properties.

![navigation component drag and drop](images/grid-view/grid-view-drag-drop-from-component-to-property.gif "navigation component drag and drop")

You can also edit the page, filter the list and get back to edited page context.

![navigation component drag and drop](images/grid-view/grid-view-navigating-component.gif "navigation component drag and drop")

## Enhanced Properties

Selecting content by “Select content” dialog is quite hard when container has more than 30 items.

![property content reference](images/grid-view/grid-view-selecting-a-content-from-tree.png "property content reference")

There are versions for ContentReference, ContentArea and ContentReference list properties. 
They can be used by adding “GridView” UIHint:

* ContentReference

![property custom content reference](images/grid-view/grid-view-property-ContentReference.png "property custom content reference")

* ContentArea

![property custom ContentArea](images/grid-view/grid-view-property-ContentArea.png "property custom ContentArea")

* ContentReference list

![property custom ContentReference list](images/grid-view/grid-view-property-ContentReferenceList.png "property custom ContentReference list")

Properties have additional button for selecting content in Grid view. Clicking the button will show the dialog with Gri View.

![custom property selection](images/grid-view/grid-view-property-select-content.png "custom property selection")

Of course there is a support for “Allowed types” and multiselect to ContentArea and ContentReference list.
Below is a demo of adding item to ContentReference property:

![using custom properties](images/grid-view/grid-view-selecting-a-ContentReference.gif "using custom properties")

You can define starting point for properties. For example for “Featured Articles” 
we would like to show the dialog with “News” container while for “Product links” 
property from the “Products” container.

![custom properties root](images/grid-view/grid-view-different-starting-points.gif "custom properties root")

## Assets pane folder command

There is new "Show grid" command that allows to show grid view for folders 
in Assets Pane. This functionality should help with listing large number 
of blocks in one folder.

![Assets pane](images/grid-view/grid-view-blocks-command.png "Assets pane")

## GridViewOptions

GridViewOptions is using Options class to configure some of the features.
Below is a description of Options properties.

| Property  | Type | Default | Description |
|---|---|---|---|
| IsComponentEnabled | bool | true | When true, then navigation component is enabled |
| IsViewEnabled | bool | true | When true, then custom view is enabled |
| ContentContainers | string | empty | Coma-separated list of ContentReferences for Content Containers under which the Navigation Tree should not display elements |
| ChildrenConvertCommandEnabled | bool | true | When true, then convert to children command is enabled |

Example that turns on GridView for all content types:
```cs
[ModuleDependency(typeof(InitializationModule))]
public class GridConfigurableModule : IConfigurableModule
{
    public void Initialize(InitializationEngine context)
    {
    }

    public void Uninitialize(InitializationEngine context)
    {
    }

    public void ConfigureContainer(ServiceConfigurationContext context)
    {
        context.Services.Configure<GridViewOptions>(config =>
        {
            config.IsViewEnabled = true;
        });
    }
}
```
