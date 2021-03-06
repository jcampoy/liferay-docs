# Providing Portlets to Manage Requests

One of the great strengths of Liferay is the sheer number of out of the box
applications. This gives it great flexibility in what it can do, which is why
Liferay Portal is used to run many different kinds of websites around the world.
For example, if you want users to add Blog posts, you can configure the Blogs
portlet to handle those requests. 

But really, that's technology from the last decade. What if you could define a
particular function that users might want to perform and let Liferay Portal
choose an available installed app to perform that function? That way, if users
want to Blog, and you've installed your own custom-developed app for blogging
instead of Liferay's, the portal can just use yours instead? 

With Liferay Portal 7, you can do just that. You can request an app based on an
entity and action type.  Processing the entity type and action, Liferay use an
available portlet that can handle the request. This increases the flexibility
and modularity of using portlets in Liferay Portal.

In this tutorial, you'll learn how to declare an entity type and action for a
desired function, and you'll create a module that finds the correct portlet to
use based on those given parameters.

## Specifying a Desired Portlet Behavior

To find the portlet you need for your particular request, you'll use the
*Portlet Providers* framework. The first thing you'll need to do is call the
[PortletProviderUtil](https://github.com/liferay/liferay-portal/blob/master/portal-service/src/com/liferay/portal/kernel/portlet/PortletProviderUtil.java)
class and request the framework find a portlet suitable for the current task.
You can request the portlet ID or portlet URL, depending on what you prefer.
Here's an example declaration:

    String portletId = PortletProviderUtil.getPortletId(
        "com.liferay.portlet.trash.model.TrashEntry, PortletProvider.Action.VIEW);

This declaration expects two parameters: the class name of the entity type you
want the portlet to handle and the type of action. The above code requests a
portlet ID for a portlet that can view Recycle Bin entries. For an in-context
example, visit the
[PortalOpenSearchImpl.search](https://github.com/liferay/liferay-portal/blob/master/portal-impl/src/com/liferay/portal/search/PortalOpenSearchImpl.java)
method.

There are four different kinds of actions supported by the Portlet Providers
framework: `ADD`, `BROWSE`, `EDIT`, and `VIEW`. Find the portlet ID or portlet
URL (depending on your needs), and specify the entity type and action you want
the portlet to handle.

+$$$

**Note:** The `getPortletURL` methods require an additional `HttpServletRequest`
or `PortletRequest` parameter, depending on which you use. Make sure to account
for this additional parameter when using the `getPortletURL` method.

$$$

You've successfully requested the portlet ID/portlet URL of a portlet that
matches your entity and action type. The portal, however, is not yet configured
to handle this request. You'll need to create a module that can find the correct
portlet to handle the request.

1. Create a generic OSGi module using your favorite third party tool.

    <!-- If we decide to document how to create an OSGi module from scratch, we
    should point to that documentation here. At the current time, there is no
    Liferay "recommended" way of doing this. Therefore, I'm assuming that the
    reader has experience with OSGi development. Pointing to introductory OSGi
    tutorials (once available) would be very helpful here. -Cody -->

2. Create a unique package name in the module's `src` directory and create a
   new Java class in that package. To follow naming conventions, name the class
   based on the element type and action type, followed by *PortletProvider*
   (e.g., `LanguageEntryViewPortletProvider`). The class should extend the
   [`BasePortletProvider`](https://github.com/liferay/liferay-portal/blob/master/portal-service/src/com/liferay/portal/kernel/portlet/BasePortletProvider.java)
   class and implement the appropriate portlet provider interface based on the
   action type you chose your portlet to handle (e.g.,
   [ViewPortletProvider](https://github.com/liferay/liferay-portal/blob/master/portal-service/src/com/liferay/portal/kernel/portlet/ViewPortletProvider.java),
   [BrowsePortletProvider](https://github.com/liferay/liferay-portal/blob/master/portal-service/src/com/liferay/portal/kernel/portlet/BrowsePortletProvider.java),
   etc.).

3. Directly above the class's declaration, insert the following annotation:

        @Component(
            immediate = true,
            property = {"model.class.name=CLASS_NAME"},
            service = INTERFACE.class
        )

    The `property` element should match the element type you specified in your
    `getPortletID/getPortletURL` declaration (e.g.,
    `com.liferay.portal.kernel.servlet.taglib.ui.LanguageEntry`). Also, your
    `service` element should match the interface you're implementing (e.g.,
    `ViewPortletProvider.class`). You can view an example of a similar
    `@Component` annotation in the
    [RolesSelectorEditPortletProvider](https://github.com/liferay/liferay-portal/blob/master/modules/apps/roles/roles-selector-web/src/com/liferay/roles/selector/web/portlet/RolesSelectorEditPortletProvider.java)
    class.

    +$$$

    **Note:** Instead of setting your `model.class.name` to a single class, you
    can set it to all classes. An example of how to set this is listed below:

        property = {"model.class.name=" + PortletProvider.CLASS_NAME_ANY}

    This means that the portlet can provide the specified action on any entity
    in Portal. You can view an example of this in the
    [AssetBrowserPortletProvider](https://github.com/liferay/liferay-portal/blob/master/modules/apps/asset/asset-browser-web/src/com/liferay/asset/browser/web/portlet/AssetBrowserPortletProvider.java)
    class.

    $$$

4. In some cases, a default portlet is already in place to handle the entity
   and action type requested. To override the default portlet with a custom
   portlet, you can assign your portlet a higher service ranking. You can do
   this by setting the following property in your `@Component` declaration:

        property= {"service.ranking:Integer=10"}

    Make sure to replace the integer with a number that is ranked higher than
    the portlet being used by default. 

5. Specify the methods you'd like to implement. Make sure to retrieve the
   portlet ID/portlet URL that should be provided when this service is called.

Lastly, generate the module's JAR file and deploy it to your portal instance.
Now a portlet that can handle the entity and action type you specified is used
when requesting a portlet ID/URL. You can now specify portlet usage without
hardcoding a specific portlet!
