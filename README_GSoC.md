#  Joomla! Core Enhancements | GSoC'21 

![Joomla@GSoC](https://community.joomla.org/images/blogs/2021/GSoC-CJO-Blogpost-Students-2021-Feature-enhancement-Yatharth-Vyas.png)


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


## Improve Menu Module Placement

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
   * Saves the Position and Menu ID which is passed as a GET param in URL by the previous step in the User State,
   * Redirects to Module Select View where user can select the Module Type
   * After the Module Type has been selected, these state items (Position and Menu ID) is used to pre-fill the Position Name and Menu ID

Here is a video demonstration of the overall frontend flow :

https://user-images.githubusercontent.com/53610833/125059779-525ff200-e0c9-11eb-89c4-e60ccb5e19d4.mp4

