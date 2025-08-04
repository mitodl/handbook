---
parent: OpenedX
---

# Configuring themes in edX

For each of our applications, we have a theme that overrides the branding in our legacy edX pages:
- MITx Online theme: https://github.com/mitodl/mitxonline-theme
- MITx Pro theme: https://github.com/mitodl/mitxpro-theme
- Residential theme: https://github.com/mitodl/mitx-theme

1. **Navigate to the Tutor Project Root Directory**

   Run the following command to go to the Tutor root path:
   ```
   cd "$(tutor config printroot)"
   ```

2. **Clone the Theme Repository**

   Go to the following directory inside the Tutor root:
   ```
   cd env/build/openedx/themes/
   ```
   Then clone the theme repo inside this folder. 
   ```
   git clone <theme-repo-url for e.g, https://github.com/mitodl/mitxonline-theme>
   ```

3. **Apply the Theme**

   Run this command to apply your custom theme:
   ```
   tutor dev do settheme <theme-name for e.g, mitxonline-theme>
   ```
   Restart the LMS/CMS container.


4. **Run the theme watcher**

   Watch the themes folders for changes:
   ```
   tutor dev run watchthemes
   ```

## Other Settings (Not required for local setup)

The configuration above is enough to run the themes locally. Below are some of the settings required for deployments:

- `ENABLE_COMPREHENSIVE_THEMING`: When enabled, this toggle activates the use of the custom theme.
- `COMPREHENSIVE_THEME_DIRS`: A list of paths to directories, each of which will be searched for comprehensive themes.
