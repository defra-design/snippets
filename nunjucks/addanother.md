
# Storing, Displaying, and Removing Multiple Data Entries in a Table Using the GDS Prototyping Kit

In this guide, you will learn how to:

- Create a form that captures multiple data entries (such as names and addresses).
- Store these entries in the session using the prototyping kit.
- Display the stored entries in a table.
- Allow users to remove specific entries from the table.

This is useful when building prototypes that involve collecting lists of information and then displaying or managing those entries.

## Overview of Steps:

1. Set up a form for capturing names and addresses.
2. Store multiple form submissions in the session.
3. Display the stored entries in a table.
4. Implement a "Remove" link for deleting specific entries.

## 1. Setting Up the Form to Capture Entries

The first step is to create a form that allows users to submit names and addresses. Each submission will be stored in the session and displayed in a table.

### Example Form:

Create a file called `views/add-entry.html` with the following form:

```html
<form action="/add-entry" method="post">
  <fieldset class="govuk-fieldset">
	<legend class="govuk-fieldset__legend govuk-fieldset__legend--l">
	  <h1 class="govuk-fieldset__heading">
		What is your address?
	  </h1>
	</legend>
	<div class="govuk-form-group">
	  <label class="govuk-label" for="name">
		Name
	  </label>
	  <input class="govuk-input" id="name" name="name" type="text" autocomplete="name">
	</div>
	<div class="govuk-form-group">
	  <label class="govuk-label" for="address">
		Address
	  </label>
	  <input class="govuk-input" id="address" name="address" type="text" autocomplete="address">
	</div>
  </fieldset>
  {{ govukButton({
	text: "Save and continue"
  }) }}
</form>
```

## 2. Storing Form Submissions in the Session

Next, create a route in `app/routes.js` that handles form submissions. This route will store each name and address entry in an array within the session.

### Add the Route for Handling Form Submissions:

In `app/routes.js`, add the following code:

```js
router.post('/add-entry', function (req, res) {
  // Check if the session data for entries exists; if not, create an empty array
  if (!req.session.data.entries) {
	req.session.data.entries = [];
  }

  // Get the submitted name and address from the form
  const name = req.body.name;
  const address = req.body.address;

  // Add the new entry to the session array
  req.session.data.entries.push({ name: name, address: address });

  // Redirect back to the form or to the view page
  res.redirect('/view-entries');
});
```

This route:

- Checks if the entries array exists in the session. If not, it initializes an empty array.
- Retrieves the submitted name and address from the form.
- Pushes the new entry into the entries array in the session.
- Redirects the user back to the form for further submissions.

## 3. Displaying the Stored Entries in a Table

Now, create a route and template that display all the stored entries in a table format.

### Add the Route to View All Entries:

In `app/routes.js`, add the following:

```js
router.get('/view-entries', function (req, res) {
  // Get the stored entries from the session, or default to an empty array if none exist
  const entries = req.session.data.entries || [];
  
  // Render the view-entries template, passing the list of entries
  res.render('view-entries', { entries: entries });
});
```

### Create the Template to Display the Table:

In `views/view-entries.html`, create the following template:

```html
{% if entries.length > 0 %}
  <table class="govuk-table">
	<caption class="govuk-table__caption govuk-table__caption--m">Names and addresses</caption>
	<thead class="govuk-table__head">
	  <tr class="govuk-table__row">
		<th scope="col" class="govuk-table__header">Name</th>
		<th scope="col" class="govuk-table__header">Address</th>
		<th scope="col" class="govuk-table__header">Action</th>
	  </tr>
	</thead>
	<tbody class="govuk-table__body">
	  {% for entry in entries %}
	  <tr class="govuk-table__row">
		<td class="govuk-table__cell">{{ entry.name }}</td>
		<td class="govuk-table__cell">{{ entry.address }}</td>
		<td class="govuk-table__cell"><a href="/remove-entry/{{ loop.index0 }}" onclick="return confirm('Are you sure you want to remove {{ entry.name }}?');">Remove</a></td>
	  </tr>
	  {% endfor %}
	</tbody>
  </table>
{% else %}
  <p>No entries have been added yet.</p>
{% endif %}
<a href="/add-entry">Add a New Entry</a>
```

`{{ loop.index0 }}`: This Nunjucks variable provides the zero-based index for each entry in the loop.

A "Remove" link is generated for each entry, passing its index to the route that will handle its removal.

## 4. Implementing the "Remove" Link

Now, create a route that handles the removal of an entry by its index.

### Add the Route for Removing an Entry:

In `app/routes.js`, add the following route:

```js
router.get('/remove-entry/:index', function (req, res) {
  // Get the index of the entry to be removed from the URL
  const index = req.params.index;

  // Check if entries exist in the session
  if (req.session.data.entries && req.session.data.entries.length > index) {
	// Remove the entry at the specified index
	req.session.data.entries.splice(index, 1);
  }

  // Redirect back to the view-entries page
  res.redirect('/view-entries');
});
```

This route:

- Retrieves the index of the entry to be removed from the URL (`req.params.index`).
- Checks if the entries array exists and contains an entry at that index.
- Removes the entry from the array using `splice()`.
- Redirects the user back to the `/view-entries` page, where the updated table will be displayed.
