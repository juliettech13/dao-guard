# Svelte Schema Form

This is a Svelte implementation of a form generator from JSON Schema. It supports subforms, lists with reordering of items, custom renderer components, and customisable CSS skinning separating layout and look-and-feel.

JSON Schema is a powerful validation/type definition language for JSON data. See this for more information on [JSON Schema](https://cswr.github.io/JsonSchema/spec/introduction/)

## How to run this repository locally

1. Install packages
```
npm install
```

2. Run local server
```
npm run dev
```

## How to use the Svelte Schema Form

Install with npm

	npm install @restspace/svelte-schema-form

Use in a component

	<script>
	  import { SubmitForm } from '@restspace/svelte-schema-form';
	  import '@restspace/svelte-schema-form/css/layout.scss';
	  import '@restspace/svelte-schema-form/css/basic-skin.scss';

	  let schema = {
		  type: "object",
		  properties: {
			  "x": { "type": "string" }
		  }
	  };
	  let value = {};

	  const submit = (e) => {
		  alert(JSON.stringify(e.detail.value, undefined, 2));
	  }
	</script>

	<SubmitForm {schema} {value} on:submit={submit} />

The `layout.css` file creates a standard form layout. `basic-skin.css` adds a very simple skin with fonts, colours etc on top of that. Note, you'll likely need to configure your bundler to interpret importing css to make this work.

The `SubmitForm` component manages validation of the entered data using the full JSON Schema spec and renders any error messages beside the relevant components. Standard behaviour is that errors are not shown until the Submit button has been clicked at least once.

## JSON Schema support

| Feature | Support |
|---|---|
|title|This property is used to label a field or fieldset, if it's absent a conversion from camel case (myFieldName) to proper case (My Field Name) is done. Newlines in the string are converted to br tags in HTML|
|description|This property is shown beside the label as a tooltip using the HTML title attribute or as a subheading for arrays/objects.|
|readOnly|This property if present and set to `true` will disable the editor for this field, only using it for display. All children of this field will also be read only|
|type="string"|By default, renders as an input element with type="text"|
|minLength, maxLength|Supported in validation|
|pattern|Supported in validation|
|format|Support for `password`, `email`, `date`, `time` and `date-time` via HTML input type|
|type="number"|By default, renders as an input element with type="number"|
|minimum, maximum|Supported in validation|
|exclusiveMinimum, exclusiveMaximum|Supported in validation|
|enum|A field with the enum property is instead rendered as a Select element with all the enum options given|
|enumText|Custom property to supply display labels for the Select element as an array of strings, one for each enum value|
|type="object"|Every object is rendered within an HTML Fieldset element with a form field for each property|
|required|Object fields named in the `required` list have to have a value entered to be valid. Required fields have the `required` class added to their labels to enable this to be displayed.
|type="array"|Every array is rendered within an HTML Fieldset element with controls for adding, deleting, moving and duplicating array items (based on the `items` property)|
|emptyDisplay|Custom property which determines how the array displays if it has no items. `false` means don't show a header or wrapper. `true` means show the header and wrapper with no items. A string value means display this message in the wrapper.
|controls|Custom property which is a comma separated list of controls including `delete`, `duplicate`, `reorder`, `add`. Default is all these. A readOnly array has no controls.| 
|editor|Custom property that lets you pick a custom editor for a schema.|
|||

## Components

### SchemaForm
A group of based on a schema, no submit functionality

    import {SchemaForm} from "@restspace/svelte-schema-form";

	<SchemaForm
	  schema // REQUIRED: JSON schema as an object
	  value // REQUIRED: JSON value to initialise the form with, for an empty form use {}
	  uploadFiles // Map with keys as property paths to upload editors, values as File objects in those editors
	  dirty // Whether the form is dirty i.e. the user has changed it from its initial value
	  showErrors // Whether the form should show errors
	  components // Map with keys as editor names (value of editor property in schema), values as constructors of editor components
	  collapsible // Whether objects and arrays are shown with collapsers which can hide them (default = false)
	  componentContext // Data for custom editors
	/>

### SubmitForm
An HTML form with a submit button and a submit flow

    import {SubmitForm} from "@restspace/svelte-schema-form";

	<SubmitForm
	  schema // REQUIRED: JSON schema as an object
	  value // REQUIRED: JSON value to initialise the form with, for an empty form use {}
	  uploadFiles // Map with keys as property paths to upload editors, values as File objects in those editors
	  uploadBaseUrl // A base URL to which uploaded files are PUT on submit. Subpaths are chosen to be unique.
	  uploadNamePattern // A substition pattern for the unique name of this record, e.g. this might be '${email}' for a person's details with a property/field named `email`
	  dirty // Whether the form is dirty i.e. the user has changed it from its initial value
	  components // Map with keys as editor names (value of editor property in schema), values as constructors of editor components
	  collapsible // Whether objects and arrays are shown with collapsers which can hide them (default = false)
	  submitText // Text shown in the submit button (defaults to 'Submit')
	  submitRequiresDirty // Whether the submit button requires the form to be dirty to submit (default = true)
	  componentContext // Data for custom editors
	/>

## Custom editors

### Currency

Setting `editor="currency"` on a `type="number"` subschema renders it as a currency field. This automatically inserts a currency symbol in the input field, by default this is a `$` but can be changed to another symbol via setting the `currencySymbol` property on the object passed in to the `componentContext` prop, to e.g. '£'. You can also specify a custom formatting function which takes a value of type number and returns a string and set the `formatCurrency` property on `componentContext` to this function.

### Radio

Setting `editor="radio"` on a subschema with `enum` set renders the enum as radio buttons instead of a select.

### Upload

Setting `editor="upload"` on a `type="string"` subschema means that this property will be rendered as a file uploader. This component requies a `SubmitForm`. The file uploader allows files to be dragged onto it, or to be clicked to open a file dialog. Files are sent on submit via a PUT request to a path composed:

    <uploadBaseUrl>/<uploadName>/<path>/<filename of file>

- `uploadBaseUrl` is the prop on `SubmitForm`
- `uploadName` is the `uploadNamePattern` with form field value(s) substituted in for `${<property path>}` codes so as to create a name that will be unique for each stored record
- `path` is the property path of the upload control with dot separators and array poitions just being an index with no square brackets e.g. `profilePics.3.image`
  
After successful submit, the property value for the editor is set to the url where the file was uploaded. The uploader has a button at the right bottom to switch modes to show a text input field with the url of the stored file.

### Autocomplete

Setting `editor="autocomplete"` on a `type="string"` subschema means this property will be rendered as an autocomplete dropdown. The autocomplete component is a text box which on each keystroke sends the entered text to a remote url to get a list of matching items which are then shown as a dropdown. The user can choose one of the displayed items at any point.

The schema needs an additional property `url` set to the base url for remote querying. Example subschema:

    {
		"type": "string",
		"editor": "autocomplete",
		"url": "https://mysite.com/autocompletes"
	}


To this url is added the query string item `match=xyz` where xyz is the current text in the search box to match on. A `GET` request is made to the result. The url should respond with an `application/json` body which can either be

    [ "dropdown item 1", "dropdown item 2", ... ]

or

    [
		{
			"id": "1234",
			"text": "An item title",
			"image": "https://images.com/an-image.jpg"
		}, ...
	]

In the latter case, the `id` field is returned as the value of the editor, the `text` field determines the text shown in the field, and the optional `image` url gives an image to display alongside the text.

### List Detail

Setting `editor="list-detail"` on a `type="array"` subschema whose items are `type="object"` shows the list of objects in a listing grid which when a row is selected, switches to the normal editor for the object selected. It also provides heading-click view ordering (without mutating the order of the underlying data list). It responds to the `emptyDisplay` and
`controls` custom properties defined for an array.

The `type="object"` subschema can have two optional custom schema properties:
- `headings`: an array of property names which are included as columns in the list. Defaults to all columns.
- `defaultSort`: an object with properties `field` which specifies the default heading field to sort on, and `direction` which can be `"asc"` or `"desc"` to specify the direction of the default sort.

## Custom rendering components

Svelte Schema Form can override or add rendering components at any level by supplying a map of component type names and component classes.

For example components, look in the /src/lib/editors directory. As an illustration, consider the Number.svelte default editor:

	<script lang="ts">
		import type { CommonComponentParameters } from "../types/CommonComponentParameters";
		export let params: CommonComponentParameters;
		export let schema: any;
		export let value: any;
	</script>

	<!-- event which calls pathChanged should be after all bindings so 'value' will have been updated -->
	<svelte:component this={params.components['fieldWrapper']} {params} {schema}>
		<input id={params.path.join('.')} name={params.path.join('.')}
			type="number" value={value || ''}
			disabled={schema.readOnly}
			on:input={ev => {
				let val = parseFloat(ev.currentTarget.value);
				params.pathChanged(params.path, isNaN(val) ? undefined : val);
			}}
		/>
	</svelte:component>

The component needs to have the 3 props shown in this component. `schema` and `value` are the local parts of the full schema and value which need to be rendered by this component. `params` contains a number of constant values defining the component including `params.pathChanged` which is a function that needs to be called with the property path to the value the component is rendering (this is `params.path`) and the new value when the value of the form field is changed.

Note how the editor supports the `readOnly` attribute by disabling the editor if set.

The `<svelte:component>` component wrapping the markup renders the default FieldWrapper.svelte component around the actual editor: this adds the field's standard label and error message.

An example of how to configure a custom component:

	<script>
	  import MyEditor from "./MyComponent.svelte";

	  let components = {
		  "myEditor": MyEditor
	  };
	  ...
	</script>

	<SubmitForm {schema} {value} on:submit={submit} {components} />

You'd then make use of this custom editor component in a schema like this:

	{
		"type": "object",
		"properties": {
			"myField": {
				"type": "string", 
				"editor": "myEditor"
			}
		}
	}

The key in the `components` map is matched to, in priority order, the `editor` property, then the `format` property, then the `type` property. This means you can also substitute editors for field groupings, which is more tricky. Work from the Array.svelte and Object.svelte default editor components in the repo.
