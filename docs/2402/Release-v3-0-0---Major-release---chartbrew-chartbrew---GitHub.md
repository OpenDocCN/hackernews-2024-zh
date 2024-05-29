<!--yml
category: 未分类
date: 2024-05-27 15:05:18
-->

# Release v3.0.0 - Major release · chartbrew/chartbrew · GitHub

> 来源：[https://github.com/chartbrew/chartbrew/releases/tag/v3.0.0](https://github.com/chartbrew/chartbrew/releases/tag/v3.0.0)

# v3 has arrived 🚀

> [📺 Watch a preview of v3 on YouTube](https://www.youtube.com/embed/15nAw318Vo4?si=Ha-uKlqItUiCfeXE)

## v3.0.0 Changelog

Lots have changed, so this changelog will be kept brief with the notable changes only.

### 🔥 Breaking changes

*   `REACT_APP_API_HOST` is now `VITE_APP_API_HOST`
*   `REACT_APP_CLIENT_HOST` is now `VITE_APP_CLIENT_HOST`
*   New variable `VITE_APP_CLIENT_PORT` to specify where the app runs or is served. The default value is `4018`, but you will have to change this if you run the app on a different port.
*   NodeJS v20 is required as a minimum version
*   Team roles have changed. The previous `admin` and `editor` roles are now set as `projectAdmin`. This will not allow them to create connections and datasets, so you might have to reassign the roles for your team members in the UI.

### ✨ New changes

*   `Connections` and `Datasets` can now be re-used on team-level and can access them from any dashboard.
*   **Introduced `tags` for Connections and Datasets to allow complex access across dashboards**
*   Satisfying new grid layout design for dashboards with variable height and drag-and-drop capabilities
*   Brand new UI design, colors, and improved UX across the board
*   New Connection creation wizard UI to help you get started much quicker
*   New Dataset builder with Chart preview and editor built-in
*   New dashboards can be created directly from community templates
*   Newly designed user dashboard with team focus
*   New roles separated by team and dashboard levels. Your team can have the `teamOwner` and `teamAdmin` roles, and your clients can have specific access to dashboards with their own `projectAdmin` and `projectViewer` roles.
*   New public dashboard design (themes coming soon!)
*   New Strapi plugin updates to work with v3, new dashboard layout, and team-switch function