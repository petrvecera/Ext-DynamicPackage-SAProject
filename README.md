# ExtJS App - Dynamic Package loading in Sencha Architect project

This demo guide is based on this guide [Packing loading on the Fly](https://sencha.guru/2017/04/12/package-loading/) written by Mitchell Simoens.  
So the main concept is based on the guide, I will post only necessary things. Please read more about each thing in the linked guide.

1. Create new blank project in the SA based on ExtJS 6.5 Modern and save it
2. Add `package-loader` to the requires in the app.json. We can do this directly in the SA.

- In the project inspector click on the `Library` than in the config panel find `Library Requires` and add `package-loader`.
- Save the project, you should see in the Cmd output that the package was added to your project.

![Library image](https://github.com/petrvecera/Ext-DynamicPackage-SAProject/blob/master/Images/library.png?raw=true)

3. Create the new packages
- Now we need to create the packages, we need to do this from the CLI. You can use SA shortcut, click on `View` -> `Open CLI` 
which will open the CLI in your project folder.
- Generate the packages using command:
```
sencha generate package -require Users
sencha generate package -require Groups 
sencha generate package -require Settings
```
If you don't have the sencha command installed, you can download the installer or use Sencha Cmd bundled with SA.
Using the Cmd bundled with SA would like this:
```
C:\Users\pagep\bin\Sencha\Architect\Cmd\6.5.0.180\sencha.exe generate package -require Users
```
Or you can add the folder to your path so you don't need to write the full path.

4. Add content to the packages. You will need to create a Main.js in the src folder of each package.
- The package code could be created manually or as separeted SA project.

```
// /path/to/MyApp/package/local/Groups/src/Main.js
   Ext.define('MyApp.view.groups.Main', {
       extend : 'Ext.Component',
       xtype  : 'myapp-groups-main',
   
       html    : 'This is the <strong>groups</strong> view!',
       padding : 20
   });
   
   // /path/to/MyApp/package/local/Settings/src/Main.js
   Ext.define('MyApp.view.settings.Main', {
       extend : 'Ext.Component',
       xtype  : 'myapp-settings-main',
   
       html    : 'This is the <strong>settings</strong> view!',
       padding : 20
   });
   
   // /path/to/MyApp/package/local/Users/src/Main.js
   Ext.define('MyApp.view.users.Main', {
       extend : 'Ext.Component',
       xtype  : 'myapp-users-main',
   
       html    : 'This is the <strong>users</strong> view!',
       padding : 20
   });
   ```

5. Edit the app.json, add uses there
- You can again use SA shortcut, click on `View` -> `Open Project Folder`, open app.json in your favorite text editor
- Add uses array under the requires:
```
"uses": [
    "Groups",
    "Settings",
    "Users"
],
```

So it will look like this:  
![app.json](https://github.com/petrvecera/Ext-DynamicPackage-SAProject/blob/master/Images/appjson.png?raw=true)

6. Build the packages using the command from the CLI:
```
sencha app build --dev --uses
```
- You should see the correct build of the packages.

7. Add the view code in the SA.
- Create tab panel
- Add 4 panels as tabs
- For each "package tab" add the package config, you can do this by writing `pkg` in the config panel 
and clicking on the `Add` button. 

![Add button](https://github.com/petrvecera/Ext-DynamicPackage-SAProject/blob/master/Images/AddButton.png?raw=true)

The code should look like this:
```
Ext.define('MyApp.view.MyTabPanel', {
    extend: 'Ext.tab.Panel',
    alias: 'widget.mytabpanel',

    requires: [
        'MyApp.view.MyTabPanelViewModel',
        'MyApp.view.MyTabPanelViewController',
        'Ext.Panel'
    ],

    controller: 'mytabpanel',
    viewModel: {
        type: 'mytabpanel'
    },

    items: [
        {
            xtype: 'panel',
            title: 'Normal Tab',
            items: [
                {
                    xtype: 'component',
                    html: 'Click on the other tabs to load the packages'
                }
            ]
        },
        {
            xtype: 'panel',
            pkg: 'Users',
            layout: 'fit',
            iconCls: 'x-fa fa-user',
            title: 'Users'
        },
        {
            xtype: 'panel',
            pkg: 'Groups',
            layout: 'fit',
            iconCls: 'x-fa fa-users',
            title: 'Groups'
        },
        {
            xtype: 'panel',
            pkg: 'Settings',
            layout: 'fit',
            iconCls: 'x-fa fa-cog',
            title: 'Settings'
        }
    ],
    listeners: {
        activeItemchange: 'onTabpanelActiveItemChange'
    }

});
```

8. Add the logic
- Select the tab panel, right click, select `Add Event Binding` create a ViewController event named 
`activeItemchange`.
- Select the function and add this code:
```
let tabpanel = sender;
let tab = value;
let pkg = tab.pkg;

    if (pkg) {
        if (Ext.Package.isLoaded(pkg)) {
            this.handlePackage(pkg);
        } else {
            tabpanel.setMasked({
                message: 'Loading Package...'
            });

            // let's use little timeout to demostrate the loading
            setTimeout(()=>{
                Ext.Package
                .load(pkg)
                .then(this.handlePackage.bind(this, pkg));
            }, 250);


        }
    }
```

- Add basic function by dragging it from the toolbox to the VC, name it `handlePackage` add param `pkg`
```
 let tabpanel = this.getView(),
        tab = tabpanel.child('[pkg=' + pkg + ']');

    tabpanel.setMasked(null);

    //only add item if the item isn't already added
    if (!tab.hasPkgItem) {
        tab.hasPkgItem = true;

        tab.add({
            xclass: 'MyApp.view.' + pkg.toLowerCase() + '.Main'
        });
    }
```

So the VC should look like this:
![View controller code](https://github.com/petrvecera/Ext-DynamicPackage-SAProject/blob/master/Images/VCcode.png?raw=true)

9. Save the project and preview it in the browser.

 





