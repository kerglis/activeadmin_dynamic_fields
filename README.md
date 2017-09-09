# ActiveAdmin Dynamic Fields [![Gem Version](https://badge.fury.io/rb/activeadmin_dynamic_fields.svg)](https://badge.fury.io/rb/activeadmin_dynamic_fields)

An Active Admin plugin to add dynamic behaviors to fields.

Features:

- set conditional checks on fields
- trigger some actions on other fields
- create links to load some content in a dialogs

The easiest way to show how this plugin works is looking the examples (below).

## Install

- Add to your Gemfile: `gem 'activeadmin_dynamic_fields'`
- Execute bundle
- Add at the end of your ActiveAdmin styles (_app/assets/stylesheets/active_admin.scss_):
`@import 'activeadmin/dynamic_fields';`
- Add at the end of your ActiveAdmin javascripts (_app/assets/javascripts/active_admin.js_):
`//= require activeadmin/dynamic_fields`

## Options

Options are passed to fields using *input_html* parameter:

- **data-if**: check a condition, values:
  + **checked**: check if a checkbox is checked
  + **not_checked**: check if a checkbox is not checked
  + **blank**: check if a field is blank
  + **not_blank**: check if a field is not blank
- **data-eq**: check if a field has a specific value
- **data-not**: check if a field hasn't a specific value
- **data-function**: check the return value of a custom function
- **data-target**: target css selector
- **data-action**: the action to trigger, values:
  + **hide**: hides elements
  + **slide**: hides elements (using sliding)
  + **fade**: hides elements (using fading)
  + **setValue**: set a value
  + **callback**: call a function
- **data-value**: value to set
- **data-callback**: custom function for setting a value
- **data-arg**: argument passed to the custom set function

## Examples of dynamic fields

- A checkbox that hides other fields if false (ex. model *Article*):

```rb
form do |f|
  f.inputs 'Article' do
    f.input :published, input_html: { 'data-if': 'not_checked', 'data-action': 'hide', 'data-target': '.grp1' }
    f.input :online_date, wrapper_html: { class: 'grp1' }
    f.input :position, wrapper_html: { class: 'grp1' }
  end
  f.actions
end
```

- Set another field value if a string field is blank:

`f.input :title, input_html: { 'data-if': 'blank', 'data-action': 'setValue', 'data-target': '#article_position', 'data-value': '10' }`

- Use a custom function for conditional check (*title_not_empty()* must be available on global scope):

`f.input :title, input_html: { 'data-function': 'title_empty', 'data-action': 'slide', 'data-target': '#article_description_input' }`

```js
function title_empty( el ) {
  return ( $('#article_title').val().trim() === '' );
}
```

- Call a callback function as action:

`f.input :published, input_html: { 'data-if': 'checked', 'data-action': 'callback', 'data-callback': 'set_title', 'data-args': '["[unpublished]"]' }`

```js
function set_title( args ) {
  if( $('#article_title').val().trim() === '' ) {
    $('#article_title').val( args[0] );
    $('#article_title').trigger( 'change' );
  }
}
```

- Custom function without action:

`f2.input :category, as: :select, collection: [ [ 'Cat 1', 'cat1' ], [ 'Cat 2', 'cat2' ], [ 'Cat 3', 'cat3' ] ], input_html: { 'data-function': 'on_change_category' }`

```js
function on_change_category( el ) {
  var target = el.closest( 'fieldset' ).find( '.pub' );
  target.prop( 'checked', ( el.val() == 'cat2' ) );
  target.trigger( 'change' );
}
```

## Example to open a dialog

Example with 2 models: *Author* and *Article*

Prepare the content dialog - in Active Admin Author config:

```rb
ActiveAdmin.register Author do
  # ...
  member_action :dialog do
    content  = '<dl>'
    [:name, :age, :created_at].each do |field|
      content += "<dt>#{Author.human_attribute_name(field)}:</dt><dd>#{resource[field]}</dd>"
    end
    content += '</dl>'
    render plain: content
  end
  # ...
end
```

Add a link to show the dialog - in Active Admin Article config:

```rb
ActiveAdmin.register Article do
  # ...
  show do |object|
    attributes_table do
      # ...
      row :author do
        link_to object.author.name, dialog_admin_author_path( object.author ), title: object.author.name, 'data-df-dialog': true, 'data-df-icon': true
      end
    end
  end
  # ...
end
```

The link url is loaded via AJAX before opening the dialog.

## Do you like it? Star it!

If you use this component just star it. A developer is more motivated to improve a project when there is some interest.

## Contributors

- [Mattia Roccoberton](http://blocknot.es) - creator, maintainer

## License

[MIT](LICENSE.txt)