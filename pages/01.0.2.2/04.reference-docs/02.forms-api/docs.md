---
title: Forms API
taxonomy:
    category: docs
---

These are pages which accept a GET request and return a prerendered HTML form. Forms are rendered server-side so that they can be customized for a particular user's context. For example, the "user" form will render differently if we are creating, versus updating, a user's information. All form api pages are stored in the `forms` directory. Success, error, and warning messages are sent to the [alert stream](/0.2.2/features#errors), and can be accessed via `api/user_alerts.php`.

#### `form_user.php`

Loads a customized form/panel for creating, updating, or displaying a user.

**Login required:** no

**Securable page:** yes

**GET request**

<table class="table table-bordered">
<thead><td>Variable name</td><td>Description</td></thead>
<tbody>
<tr><td><code>box_id</code></td><td>The desired <code>id</code> of the div that will contain the form.</td></tr>         
<tr><td><code>render_mode</code></td><td>Render the form as a popup (modal) window, or in a flat panel.  Set to <b>modal</b> or <b>panel</b>.</td></tr> 
<tr><td><code>[user_id]</code></td><td>If specified, will load the relevant data for the user into the form.  Used for viewing/updating a user.</td></tr>
<tr><td><code>[fields]</code></td><td>A list of fields to render.  If not specified, the following fields will be rendered by default:
  <ul>
  <li>user_name</li>
  <li>display_name</li>
  <li>email</li>
  <li>title</li>
  <li>sign_up_stamp</li>
  <li>last_sign_in_stamp</li>
  <li>groups</li>
  </ul>
  By default, they will all be read-only fields.
</td></tr>
<tr><td><code>[buttons]</code></td><td>A list of buttons to render.  If not specified, the following fields will be rendered by default:
  <ul>
  <li>btn_edit</li>
  <li>btn_activate (inactive users only)</li>
  <li>btn_enable (disabled users only)</li>
  <li>btn_disable (enabled users only)</li>
  <li>btn_delete</li>
  </ul>
</td></tr>
</tbody>
</table>

**GET response**

<p>A JSON object containing the following fields:</p>
<table class="table table-bordered">
<thead><td>Variable name</td><td>Description</td></thead>
<tbody>
<tr><td><code>data</code></td><td>The fully rendered HTML for the form/panel.</td></tr>
</tbody>
</table>

**Possible alerts generated**

- `SQL_ERROR`: Database error.
- Authorization failed
