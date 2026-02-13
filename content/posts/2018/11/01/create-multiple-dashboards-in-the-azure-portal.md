+++
date = '2018-11-01T10:12:00+01:00'
draft = false
title = 'Create multiple dashboards in the Azure Portal'
categories = ['Technology']
tags = ['Azure']
+++

Several years ago I wrote a very quick post on the Azure Portal user-experience called [Using the new Azure Dashboard favourites and tiles](https://tenbulls.co.uk/2016/02/29/using-the-new-azure-dashboard-favourites-and-tiles/) which demonstrated how Microsoft is improving the usability of the Azure Portal. As such, It is probably worth a very quick mention that I recently noticed the [Microsoft Azure Portal](https://portal.azure.com) now supports multiple dashboards, and according to my Google-Bing-Fu seems to have done so for a couple of years.

Strange that I had missed (or ignored) it until now, and because of that reason, it is probably worth me highlighting this usability feature it in case I am not alone.

# Creating dashboard

At the top of your screen, click the *Dashboard* menu item to jump to your current dashboard. Now you will see the menu panel provides the ability to create a new dashboard, import a dashboard (from a JSON document), save an existing dashboard (to a JSON document), or even share a dashboard.

![Ribbon](/images/2018/ribbon.png)

Click the *New dashboard* menu item and a new dashboard will open in edit mode. Simply provide your dashboard with a new name (in my case I will call it *Demos*), and from the *Tile Gallery* on the left-hand pane you can add and resource types required onto your new panel.

![Tile Gallery](/images/2018/blankcanvas.webp)

Each of these resource tiles should be configured simply by clicking on the *Configure tile* text for those where it is available (otherwise your resource tile will sit unconfigured until you do). This will allow you to filter this tile down to a specific resource binding, but once you have done this the binding to the tile in question cannot (at this time of writing) be altered.

![New Dashboard](/images/2018/newdash.png)

Click *Done customizing* to create your new panel (you can re-edit this at any time) and you can now switch quickly between dashboards using the *Dashboard* dropdown.

# Pinning resources

In addition to adding tiles to the dashboard through the *Tile Gallery*, you can also pin a filtered resource group to your currently selected dashboard simply by navigating to a resource type, selecting those you require and clicking the *Pin blade to dashboard icon* found in the top right of your screen. Instead, you might prefer to pin individual resources to the dashboard simply by drilling down into a resource and hitting the pin icon like before. Any pinned items will be pinned to the dashboard in view.

![Pin blade](/images/2018/pinblade.png)

Once you are happy with your dashboard, you might find it useful to make a backup of this view by hitting the *Download* menu item (from your dashboard view). This will simply save a local JSON file that describes your dashboard configuration. If you delete your existing dashboard and want to restore it, all you need to do is import the JSON file (through the *Upload* menu item) and a new dashboard will be created from this template. It is possible to import this file as many times as you want since the Azure Portal will auto-name new dashboards if there are any conflicts. Azure Portal will prevent you from deleting all dashboards (so you must at least have one).

---

# Summary

While most of us might only ever need a single dashboard view, I can see lots of potential benefits to creating multiple dashboards as our Cloud estate grows and we need to limit what we see. This could be very useful for delivering clearer demos to your audience and will prevent you from constantly fiddling with your current dashboard. You can very easily backup and restore dashboards should you make a mistake, so there is really no reason why you shouldn't investigate what other UX improvements are available to you in the Azure Portal!