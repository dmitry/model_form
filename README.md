# model_form

In a big or even medium project fat controllers and models can become a real problem.
With that gem you can move specific form parts of your logic to a `forms`.


## Features

* callbacks and validations
* virtual attributes
* nested attributes (can be replacements for a `accepts_nested_attributes_for` in your model)
 * with correct error messages mapped to the nested attributes
 * without `_attributes` postfix
* transaction safe (same as autosave)

## Future support

* multi-form - one form may consists of > 1 model
* orm agnostic: activerecord, mongoid
* multiparameter attributes

## Inspiration

Related: PORO, DAO, Service Object

Frameworks:

* Django - ModelForms
* Symfony - Forms (http://symfony.com/doc/current/book/forms.html#index-15)

Articles:

* http://pivotallabs.com/users/jdean/blog/articles/1706-form-backing-objects-for-fun-and-profit
* http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/
* http://www.devcaffeine.com/2012/06/20/isolating-validations-in-activemodel/
* http://fredwu.me/post/54009567748/datamappify-a-new-take-on-decoupling-domain-form-and
* http://brainspec.com/blog/2012/07/09/activerecord-inheritance-contexts/ - context technique
* http://rubyflow.ru/items/1481 - in russian language about `form_logic` gem

Videos:

* http://railscasts.com/episodes/16-virtual-attributes
* http://railscasts.com/episodes/398-service-objects

Gems:

* https://github.com/orgsync/active_interaction - prefer that over everything else
* https://github.com/cypriss/mutations - more or less that gem would be
* https://github.com/fredwu/datamappify
* https://github.com/hookercookerman/form_model
* https://github.com/apotonick/reform
* https://github.com/ClearFit/redtape - nested attributes
* https://github.com/MSch/activemodel-form
* https://github.com/saratovsource/form_object
* https://github.com/joevandyk/cat_forms
* https://github.com/brycesenz/freeform
* https://github.com/braintree/curator
* https://github.com/nulogy/edr
* https://github.com/joakimk/minimapper
* https://github.com/GCorbel/activeform-rails
* https://github.com/shlima/form_logic

Additional readings:

* https://github.com/rails/rails/pull/8189 - rails pull request for refactoring multi-parameter to a controller; nice idea to have it in this gem

## Concept example
    
```ruby
class Article < ActiveRecord::Base
  has_many :photos
end

class Photo < ActiveRecord::Base
  # belongs_to :article
end


class ArticleForm < ModelForm
  attr_accessible :header, :description, :photos

  attribute :accepted, Boolean
  
  after_assign do
    self.description = translate(description, :en, :ru)
    
    # triggers after_assign in nested attribute
  end
  
  before_validaiton do
    # runs after_assign
  end
  
  validates :name, presence: true
  
  # no photos_attributes - think about it, because ActiveRecord and ActionPack have some references on _attributes
  nested_attributes :photos
  
  def accepted=(accepted)
    self.accepted_at = accepted ? Time.now : nil
  end
  
  class PhotoForm # < NestedForm
    
  end
  
  private
  
  def translate(text, from, to)
    # translation logic here
  end
end

article_form = ArticleForm.new(header: 'Header name', description: 'Some description here', photos: [{id: 1, name: 'some name'}, {name: 'new'}])
article_form.assign_attributes header: 'Update header name'
article_form.save # or save!
article_form.valid?
article_form.invalid?
article_form.errors
```

## API

### Callbacks

##### assign_attributes

`before_assign`

`after_assign`

##### save

`before_validaiton`

`before_save`

`after_save`

`after_commit`

### Class

`attr_accessible` may take block or list of attributes to whitelist

`attribute(name, type)` add virtual attribute to a form with specific type

[`nested_attributes(association_name, options={})`](#nested_attributes) add ability to set nested attributes for some association

### Instance

`assign_attributes`

`valid?`

`invalid?`

`save`

`save!`

`model`

`params`

`errors` - TODO think how it should be: nested or plain!?

## Extended API description

##### nested_attributes

* `association_name`: association name that used in one of association types
* `options` by default is `{}`
 * `class_name` by default "#{attribute_name.to_s.singularize.camelcase}Form", but can be a model or other ModelForm
 * `reject_if`
 * `allow_destroy`

## What else?!

From the beginning I've tried to develop this gem with `RDD` (Readme Driven Development - http://tom.preston-werner.com/2010/08/23/readme-driven-development.html)
As you can see, it not works yet, but the idea lives.
