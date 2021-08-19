#  Joomla! Core Enhancements | GSoC'21 

![Joomla GSoC Banner](https://community.joomla.org/images/blogs/2021/GSoC-CJO-Blogpost-Students-2021-Feature-enhancement-Yatharth-Vyas.png)


## Introduction

Reference: [Project Idea Link](https://docs.joomla.org/GSoC_2021_Project_Ideas#Project_III:_Feature_enhancement)

Core Enhancements covers 4 tasks:

1. [Improve Menu Module Placement](#improve-menu-module-placement)
2. [Improve the incorporation of Modules in Articles](#improve-the-incorporation-of-modules-in-articles)
3. [Integrate Workflows in Modules Component](#integrate-workflows-in-modules-component)
4. [Merge Featured Articles into Articles](#merge-featured-articles-into-articles)

<br>
Along with these, there was 1 bonus task:

- [Display Modules Position Badge Background Color based on the Active Template Positions](#modules-position-badge)

<br>

## Pull Requests

| Task                                             | Project Repo PR                                                               | Joomla CMS Repo PR                                           |
|--------------------------------------------------|-------------------------------------------------------------------------------|--------------------------------------------------------------|
| Improve Menu Module Placement                    | [PR #1](https://github.com/joomla-projects/gsoc21_core-enhancements/pull/1)   | [PR #34743](https://github.com/joomla/joomla-cms/pull/34743) |
| Improve the incorporation of Modules in Articles | [PR #8](https://github.com/joomla-projects/gsoc21_core-enhancements/pull/8)   | [PR #34764](https://github.com/joomla/joomla-cms/pull/34764) |
| Integrate Workflows in Modules Component         | [PR #10](https://github.com/joomla-projects/gsoc21_core-enhancements/pull/10) | [PR #35101](https://github.com/joomla/joomla-cms/pull/35101) |
| Merge Featured Articles into Articles            | [PR #11](https://github.com/joomla-projects/gsoc21_core-enhancements/pull/11) | [PR #35228](https://github.com/joomla/joomla-cms/pull/35228) |
| Bonus - Modules Position Badge                   | [PR #3](https://github.com/joomla-projects/gsoc21_core-enhancements/pull/3)   | Coming Soon                                                  |

<hr>

## Improve Menu Module Placement

PR Link: https://github.com/joomla/joomla-cms/pull/34743

This task re-invents the conventional method of placing modules from the Backend Dashboard to an easier to visualize series of steps where the users can visit a particular page in the Frontend (while logged in to an admin account), and from there, they can select the menu (automatically determined based on the page) and position of the Module by means of buttons that reflect the output of where the module will be placed if the user proceeds to create it.

This task has 2 parts:

1. [Frontend Flow](#frontend-flow)
2. [Backend Flow](#backend-flow)

### Frontend Flow

- **New Plugin**
    * Introduce a new system plugin `plg_system_addmodulebutton` for displaying the Add Module Button
    * This button is appended to the end of the `<main>` tag which is omnipresent on all pages. 
    * This button is only displayed when a user logs into the frontend using an account that has Module Add/Edit Permissions
    * Pressing this button adds `?pm=1` as a GET parameter in the URL. This is similar to the `?tp=1` parameter that is used for displaying [template positions](https://docs.joomla.org/Finding_module_positions_on_any_given_page)


- **Place Module** (`?pm=1`) **View** 
    * This view displays the current page will all possible template positions
    * Each position will have a button corresponding to it. This button can be used to select this position for placing the new module.
    * These buttons (to select position) redirect the user to a backend page having a URL like `administrator/index.php?option=com_modules&task=module.selectPosition&position=footer&menu=:id` . This URL has 3 parts:
        1. `task=module.selectPosition` is used to call the selectPosition Controller in com_modules/module
        2. `&position=footer` is used to pass the position as a GET param
        3. `&menu=:id` is used to pass the menu id of the current page as a GET param

- **New Controller: /com_modules/Module/selectPosition**
   * Saves the Position and Menu ID which is passed as a GET param in URL by the previous step in the User State <br> <br>
    ```php
    $app->setUserState('com_modules.add.module.menu_id', $menuId);
    $app->setUserState('com_modules.add.module.position', $position);
    ```
   * Redirects to Module Select View where user can select the Module Type
   * After the Module Type has been selected, these state items (Position and Menu ID) is used to pre-fill the Position Name and Menu ID


Here is a video demonstration of the overall frontend flow :

https://user-images.githubusercontent.com/53610833/125059779-525ff200-e0c9-11eb-89c4-e60ccb5e19d4.mp4


### Backend Flow

Editing Modules in the Backend Admin Panel turned out to be a little tricky for the user if the template positions were not easy to visualize or if the User was unaware of the layout of the template positions. To simplify all this into an easier and efficient method, a new modal has been added to select the positions via their preview. The video below demonstrates this new modal:

https://user-images.githubusercontent.com/53610833/125062858-783ac600-e0cc-11eb-9955-9fb4aa60c897.mp4

<hr>

## Improve the incorporation of Modules in Articles

PR Link: https://github.com/joomla/joomla-cms/pull/34764

This task has 2 parts:

1. [Create Modules in Article Edit View](#create-modules-in-article-edit-view)
2. [Imported Modules Tab](#imported-modules-tab)

### Create Modules in Article Edit View

Users can add Modules in Articles in Joomla. This is done with the help of the Module xtd-editor Plugin. However, this Plugin opens a Modal that only allows the user to select a pre-existing module, ie. there is no option to add a new module directly in the Article Edit View.
To enhance that, this PR introduces a button to Create Modules directly in the Article Edit View:

https://user-images.githubusercontent.com/53610833/125674142-803bf23a-b3a9-47e4-b6b1-8013e40d9693.mp4

This is implemented using a cookie which can be read and modified by both PHP (to not display the Save and Close button in Toolbar for unsaved modules) and JavaScript (to clear this flag when we close the modal). Hiding Save and Close button is necessary here because we need to make sure that the module is saved (in our database) before we close the modal so we can grab the module's ID.

This task has been implemented by re-using the existing controller functions by modifying them just enough to support backward compatibility. 


### Imported Modules Tab

This tab allows us to perform Edit and Remove operations on the imported module using buttons. It displays Modules that are imported using any of the 3 possible syntaxes (supports older import syntaxes for backwards compatibility):

1. `{loadmoduleid :id}`
2. `{loadmodules :mod_type}`
3. `{loadposition :position, :card_style}`

These are matched using regex:

```php
// Expression to search for (positions)
$regex = '/{loadposition\s(.*?)}/i';

// Expression to search for (modules)
$regexmod = '/{loadmodule\s(.*?)}/i';

// Expression to search for (id)
$regexmodid = '/{loadmoduleid\s([1-9][0-9]*)}/i';
```

![Imported Modules Tab](https://user-images.githubusercontent.com/53610833/126033410-9581b90e-0583-4d95-893c-e730daa899fd.png)

<hr>

## Integrate Workflows in Modules Component

PR Link: https://github.com/joomla/joomla-cms/pull/35101


### Access
- The corresponding access.xml actions have been added.

### Articles Model
- The `getListQuery()` method joins the old query with workflow_associations to also consider the workflow stages that belong to `com_content.articles` only.

### SQL Changes:
- The following have been added to the Database Tables:
    - 1 new workflow: `COM_WORKFLOW_BASIC_WORKFLOW_MODULES`
    - 1 new stage:  `COM_WORKFLOW_BASIC_STAGE`
    - 3 new transitions: Publish, Unpublish, and Trash
- Workflow Associations have been created for all existing modules via an update script / a query during Installation.

### Introduce a new Backend Menu Item
- A new menu item for Modules: Workflows to execute CRUD for `com_modules`

### Option to Enable Workflow Integration
- This can be toggled from Global Configuration

### Searchtools Filtering
- Option to filter modules via their stage

### ModulesComponent.php
- Added necessary functions that are used by Workflow Plugins (Workflows, Featuring and Publishing)

### Module Model
- The get query now takes into account the workflow stage associated with each module.
- Added Workflow before and after Save events
- Added a function to get the initial stage for a module (default workflow and default stage)
- Added a query to create workflow association while duplicating modules
- Added a function that prepares the workflow field

### Modules Model
- Add a filterForm function to filter based on workflow stage
- Join the query with workflow associations
```php
// Join over the workflows association
$query->select($db->quoteName('wa.stage_id', 'stage_id'))
	->join('INNER', $db->quoteName('#__workflow_associations', 'wa'), $db->quoteName('wa.item_id') . ' = ' . $db->quoteName('a.id'))
	->where($db->quoteName('wa.extension') . ' = ' . $db->quote('com_modules.module'));

// Join over the workflows stage
$query->select(
	[
		$db->quoteName('ws.title', 'stage_title'),
		$db->quoteName('ws.workflow_id', 'workflow_id')
	])
	->join('INNER', $db->quoteName('#__workflow_stages', 'ws'), $db->quoteName('ws.id') . ' = ' . $db->quoteName('wa.stage_id'));

// Join over the workflows
$query->select($db->quoteName('w.title', 'workflow_title'))
	->join('INNER', $db->quoteName('#__workflows', 'w'), $db->quoteName('w.id') . ' = ' . $db->quoteName('ws.workflow_id'));
```

### Modules Template:
- Added workflows column to the table
- This column is conditionally rendered when workflows are enabled
- Added workflows field to default batch body

### The Flow:
1. When creating new modules you have to select a workflow and when you save, the default stage under this workflow_id is set as the initial stage of the module.
2. When editing exiting modules, the workflow transition field is shown instead of the workflow_id field, which works similarly to com_content.

#### New Modules: Select Workflow 

(Permanent similar to workflow assignment for com_content)

![image](https://user-images.githubusercontent.com/53610833/127371792-76191d17-2556-4147-be8d-6e95f835f657.png)


#### Edit Existing Modules: Transition Field

![image](https://user-images.githubusercontent.com/53610833/127372084-a33fff64-3a05-4189-a00e-0d6ac782b6f9.png)

<hr>

## Merge Featured Articles into Articles

PR Link: https://github.com/joomla/joomla-cms/pull/35228

The primary aim for this task was to refactor the code in order to enhance the code re-usability and eliminate redundancy.
<br>

### [A] New Selector Dropdown
A searchtools selector is added that can be used to toggle between:

Option | Value
------------ | -------------
All  | ""
Unfeatured  | 0
Featured  | 1

![dropdown](https://user-images.githubusercontent.com/53610833/128350490-c5d83ab1-2ffb-42f7-8ecc-27cde5d658c5.gif)

<br>

### [B] Model Merged
- A new method to return the featured selector filter parameter using the getUserState from request
```php
$featured = $this->getUserStateFromRequest($this->context . '.featured', 'featured', '');
```
- The advantage of the above is that we can also pass the value as a GET param to the URL which helps us to make redirect URLs that can initialize the value of this dropdown
- `$featured` variable is used as a condition to manipulate the query to adjust to Featured

<br>

### [C] Templates Merged
- We use `state->get('featured')` to conditonally render the template code as per featured or not.

<br>

### [D] Views Merged
- We use `state->get('featured')` to conditonally render the toolbar options

<br>

### [E] Controller Merged
- Merged the delete function from FeaturedController

<br>
