# Nav Menu Roles #

**Contributors:** helgatheviking  
**Donate link:** https://www.paypal.me/helgatheviking  
**Tags:** menu, menus, nav menu, nav menus  
**Requires at least:** 4.4.0  
**Tested up to:** 4.4.0  
**Stable tag:** 1.7.9
**License:** GPLv3  

Hide custom menu items based on user roles. PLEASE READ THE [FAQ](#frequently-asked-questions) IF YOU ARE NOT SEEING THE SETTINGS.

## Description ##

This plugin lets you hide custom menu items based on user roles.  So if you have a link in the menu that you only want to show to logged in users, certain types of users, or even only to logged out users, this plugin is for you.

Nav Menu Roles is very flexible. In addition to standard user roles, you can customize the functionality by adding your own check boxes with custom labels using the `nav_menu_roles` filter and then using the `nav_menu_roles_item_visibility` filter to check against whatever criteria you need. You can check against any user meta values (like capabilities) and any custom attributes added by other plugins. See the [FAQ](#im-using-xyz-membership-plugin-and-i-dont-see-its-levels).

### IMPORTANT NOTE ###

In WordPress menu items and pages are completely separate entities. Nav Menu Roles does not restrict access to content. Nav Menu Roles is *only* for showing/hiding *nav menu* items. If you wish to restrict content then you need to also be using a membership plugin.

### Usage ###

1. Go to Appearance > Menus
1. Edit the menu items accordingly.  First select whether you'd like to display the item to all logged in users, all logged out users or to customize by role.
**1. If you chose customize by role, keep in mind that the role doesn't limit the item strictly to that role, but to everyone who has that role's capability. For example:** an item set to "Subscriber" will be visible by Subscribers *and* by admins. Think of this more as a minimum role required to see an item.   
1. If you choose 'By Role' and don't check any boxes, the item will be visible to everyone like normal.

### Support ###

Support is handled in the [WordPress forums](https://wordpress.org/support/plugin/nav-menu-roles). Please note that support is limited and does not cover any custom implementation of the plugin. Before posting, please read the [FAQ](http://wordpress.org/plugins/nav-menu-roles/faq/). Also, please verify the problem with other plugins disabled and while using a default theme. 

Please report any bugs, errors, warnings, code problems to [Github](https://github.com/helgatheviking/nav-menu-roles/issues)

## Installation ##

1. Upload the `plugin` folder to the `/wp-content/plugins/` directory
1. Activate the plugin through the 'Plugins' menu in WordPress
1. Go to Appearance > Menus
1. Edit the menu items accordingly. First select whether you'd like to display the item to Everyone, all logged out users, or all logged in users. 
1. Logged in users can be further limited to specific roles by checking the boxes next to the roles you'd like to restrict visibility to.

## Screenshots ##

### 1. Show the new options for the menu items in the admin menu customizer ###
![Show the new options for the menu items in the admin menu customizer](http://plugins.svn.wordpress.org/nav-menu-roles/assets/screenshot-1.png)


## Frequently Asked Questions ##

# I don't see the Nav Menu Roles options in the admin menu items?  #

This is because you have another plugin (or theme) that is also trying to alter the same code that creates the Menu section in the admin.  

WordPress does not have sufficient hooks in this area of the admin and until they do plugins are forced to replace everything via custom admin menu Walker, of which there can be only one. There's a [trac ticket](http://core.trac.wordpress.org/ticket/18584) for this, but it has been around a while. 

**A non-exhaustive list of known conflicts:**

1. UberMenu 2.x Mega Menus plugin
2. Add Descendants As Submenu Items plugin
3. Navception plugin
4. Suffusion theme
5. BeTheme
6. Yith Menu
7. Jupiter Theme


# Workaround #1 #
[Shazdeh](https://profiles.wordpress.org/shazdeh/) had the genius idea to not wait for a core hook and simply add the hook ourselves. If all plugin and theme authors use the same hook, we can make our plugins play together.

Therefore, as of version 1.6 I am modifying my admin nav menu Walker to *only* adding the following lines (right after the description input):

```
<?php 
// Place this in your admin nav menu Walker
do_action( 'wp_nav_menu_item_custom_fields', $item_id, $item, $depth, $args );
// end added section 
?>
```

**Ask your conflicting plugin/theme's author to add this code to his plugin or theme and our plugins will become compatible.**

# Patching Your Plugin/Theme #

**Should you wish to attempt this patch yourself, you can modify your conflicting plugin/theme's admin menu Walker class. **Reminder:** I do not provide support for fixing your plugin/theme. If you aren't comfortable with the following instructions, contact the developer of the conflicting plugin/theme!**  

1. Find the class that extends the `Walker_Nav_Menu`. As a hint, it is filtering `wp_edit_nav_menu_walker` and you might even be getting a warning about it from Nav Menu Roles. Example: 

```
add_filter( 'wp_edit_nav_menu_walker', 'sample_edit_nav_menu_walker');
function sample_edit_nav_menu_walker( $walker ) {
    return 'Walker_Nav_Menu_Edit_Roles'; // this is the class name
}
```

2. Find the file for the extending class. In my plugin this is in a file located at `inc/class.Walker_Nav_Menu_Edit_Roles.php`. I can't know *where* this file is in your plugin/theme. Please don't ask me, but here's what the beginning of that class will look like:

`class Walker_Nav_Menu_Edit_Roles extends Walker_Nav_Menu {}`

Note that the class name is the same as the class name you found in step 1.

3. Find the `start_el()` method

In that file you will eventually see a class method that looks like:

`function start_el( &$output, $item, $depth = 0, $args = array(), $id = 0 ) {`

4. Paste my action hook somewhere in this method!

In Nav Menu Roles, I have placed the hook directly after the description, ex:

```
<p class="field-description description description-wide">
  <label for="edit-menu-item-description-<?php echo $item_id; ?>">
    <?php _e( 'Description' ); ?><br />
    <textarea id="edit-menu-item-description-<?php echo $item_id; ?>" class="widefat edit-menu-item-description" rows="3" cols="20" name="menu-item-description[<?php echo $item_id; ?>]"><?php echo esc_html( $item->description ); // textarea_escaped ?></textarea>
    <span class="description"><?php _e('The description will be displayed in the menu if the current theme supports it.'); ?></span>
  </label>
</p>

<?php 
// Add this directly after the description paragraph in the start_el() method
do_action( 'wp_nav_menu_item_custom_fields', $item_id, $item, $depth, $args );
// end added section 
?>
```

### Workaround #2 ###

As a workaround, you can switch to a default theme (or disable the conflicting plugin), edit the Nav Menu Roles, for each menu item, then revert to your original theme/ reenable the conflicting plugin. The front-end functionality of Nav Menu Roles will still work. 

### I'm using XYZ Membership plugin and I don't see its "levels"? ###

There are apparently a few membership plugins out there that *don't* use traditional WordPress roles/capabilities. My plugin will list any role registered in the traditional WordPress way. If your membership plugin is using some other system, then Nav Menu Roles won't work with it out of the box.  Since 1.3.5 I've added a filter called `nav_menu_roles_item_visibility` just before my code decides whether to show/hide a menu item. There's also always been the `nav_menu_roles` filter which lets you modify the roles listed in the admin. Between these two, I believe you have enough to integrate Nav Menu Roles with any membership plugin. 

Here's an example where I've added a new pseudo role, creatively called "new-role".  The first function adds it to the menu item admin screen. The second function is pretty generic and won't actually do anything because you need to supply your own logic based on the plugin you are using.  Nav Menu Roles will save the new "role" info and add it to the item in an array to the `$item->roles` variable.

# Adding a new "role" #

```
/*
 * Add custom roles to Nav Menu Roles menu list
** * param:** $roles an array of all available roles, by default is global $wp_roles   
** * return:** array  
 */
function kia_new_roles( $roles ){
  $roles['new-role-key'] = 'new-role';
  return $roles;
}
add_filter( 'nav_menu_roles', 'kia_new_roles' );
```

Note, if you want to add a WordPress capability the above is literally all you need. Because Nav Menu Roles checks whether a role has permission to view the menu item using `current_user_can($role) you do not need to right a custom callback for the `nav_menu_roles_item_visibility` filter.

In case you *do* need to check your visibility status against something very custom, here is how you'd go about it:

```
/*
 * Change visibilty of each menu item
** * param:** $visible boolean  
** * param:** $item object, the complete menu object. Nav Menu Roles adds its info to $item->roles  
 * $item->roles can be "in" (all logged in), "out" (all logged out) or an array of specific roles
 * return boolean
 */
function kia_item_visibility( $visible, $item ){
  if( isset( $item->roles ) && is_array( $item->roles ) && in_array( 'new-role-key', $item->roles ) ){
  /*  if ( // your own custom check on the current user versus 'new-role' status ){
        $visible = true;
      } else {
        $visible = false;
    }
  */  }
  return $visible;
}
add_filter( 'nav_menu_roles_item_visibility', 'kia_item_visibility', 10, 2 );
```

Note that you have to generate your own if/then logic. I can't provide free support for custom integration with another plugin. You may [contact me](http://kathyisawesome.com/contact) to discuss hiring me, or I would suggest using a plugin that supports WordPress' roles, such as Justin Tadlock's [Members](http://wordpress.org/plugins/members).

### The menu exploded? Why are all my pages displaying for logged out users? ###

If every item in your menu is configured to display to logged in users (either all logged in users, or by specific role), then when a logged out visitor comes to your site there are no items in the menu to display.  `wp_nav_menu()` will then try check its `fallback_cb` argument... which defaults to `wp_page_menu`.

Therefore, if you have no items to display, WordPress will end up displaying ALL your pages!!

If you don't want this, you must set the fallback argument to be a null string.

```
wp_nav_menu( array( 'theme_location' => 'primary-menu', 'fallback_cb' => '' ) );
```

### What happened to my menu roles on import/export? ###

The Nav Menu Roles plugin stores 1 piece of post *meta* to every menu item/post.  This is exported just fine by the default Export tool.

However, the Import plugin only imports certain post meta for menu items.  As of version 1.3, I've added a custom Importer to Nav Menu Roles as a work around.

### How Do I Use the Custom Importer? ###

1. Go to Tools>Export, choose to export All Content and download the Export file
1. Go to Tools>Import on your new site and perform your normal WordPress import
1. Return to Tools>Import and this time select the Nav Menu Roles importer.
1. Use the same .xml file and perform a second import
1. No duplicate posts will be created but all menu post meta (including your Nav Menu Roles info) will be imported

