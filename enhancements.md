# Feature: Fine-Grained Page Permissions & Structured UI

## Summary
This pull request introduces **Fine-Grained Page Permissions** to Filament Shield. Previously, Filament pages were restricted to a single "view" permission. This update allows developers to define multiple custom permissions per page, provides a structured UI in the Role Resource, and maintains strict namespace separation to avoid collisions with Resource permissions.

---

## Changes

### 1. Configuration & Namespacing
* **Permission Prefix:** Added a new configuration key `pages.permission_prefix` (defaulting to `'page'`) to ensure page permissions are clearly distinguished from resource permissions in the database.
* **Three-Part Convention:** Enforced a naming convention: `{Prefix}{Separator}{Action}{Separator}{Subject}` (e.g., `Page:InsertSick:HolidayMatrixTable`), strictly respecting the global `permissions.separator` and `permissions.case` settings.

### 2. Core Logic Enhancements (`FilamentShield.php`)
* **Discovery:** Updated `getDefaultPermissionKeys()` to detect the new static method `getShieldPagePermissions()` on Page classes.
* **Standardized Structure:** Refactored the internal data structure for Pages and Widgets to return an array of `['key' => string, 'label' => string]`.
* **Fallback Safety:** Unified the fallback logic to ensure even standard pages follow the new structured format, preventing "ghost permissions" (naked prefixes like `view`).

### 3. Generator & Support Logic
* **Collection Mapping:** Refactored `GenerateCommand.php` (`generateForPages` & `generateForWidgets`) to iterate through structured permission arrays instead of assuming a single key.
* **Ghost Permission Fix:** Modified `Utils.php` to ensure the generator never creates a permission without its entity context.

### 4. Role Resource UI (`HasShieldFormComponents` Trait)
* **Section-Based Layout:** Replaced the flat checkbox list for pages with a **Grid of Sections**, mirroring the "Resources" tab layout for better UX.
* **Class Namespaces:** Added the full **Page Class Namespace (FQCN)** as a description within each section to assist developers in identifying pages with similar titles.
* **Type Safety:** Fixed a type error in `CheckboxList` rendering by ensuring options are correctly flattened into `key => label` pairs.

### 5. Frontend Integration (`HasPageShield` Trait)
* **Fluent Helper:** Introduced `$this->canShield('action')` to allow easy, type-safe permission checks within Page classes and Blade templates.
* **Smart Access Control:** Updated `canAccess()` to use `hasAnyPermission()`. Users can now access a page if they possess **any** of the defined permissions for that page, facilitating partial access scenarios.

---

## Implementation Example

To implement fine-grained permissions, add the following method to your Filament Page:

```php
public static function getShieldPagePermissions(): array
{
    return ['view', 'editAllSettings', 'importZeusExcel', 'insertSick'];
}


// In the Page Class
if ($this->canShield('insertSick')) {
    // Perform restricted action
}

// In the Blade View
@if($this->canShield('editAllSettings'))
    <x-filament::input wire:model="settings.global_value" />
@endif
