model_form
==========

Features
========

* additional callbacks and validations
* virtual attributes
* 

Goals
=====

* nested - extract or make an alternative of `accepts_nested_attributes_for`
* multi-form - one form may consists of > 1 model
* transaction safe
* 

Inspiration
===========

Related: PORO, DAO, ServiceLayer

Frameworks:

* Django - ModelForms
* Symfony - Forms (http://symfony.com/doc/current/book/forms.html#index-15)

Articles:

* http://pivotallabs.com/users/jdean/blog/articles/1706-form-backing-objects-for-fun-and-profit
* http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/
* http://www.devcaffeine.com/2012/06/20/isolating-validations-in-activemodel/

Gems:

* https://github.com/MSch/activemodel-form
* https://github.com/joevandyk/cat_forms
* https://github.com/ClearFit/redtape

Additional readings:

* https://github.com/rails/rails/pull/8189 - rails pull request for refactoring multi-parameter to a controller; nice idea to have it in this gem

Concept example
===============
    
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

API

`attr_accessible`

may take block or list of attributes to whitelist

`attribute(name, type)`

add virtual attribute to a form with specific type

`assign_attributes`


`after_assign`

callback takes block and invokes after `assign_attributes`

`before_validaiton`

callback takes block and invokes after `after_assign` and before validations

`validates`

callback takes block and invokes after all the validations invoked

`nested_attributes(association_name, options={})`

* `association_name`: association name that used in one of association types
* options:
* `class_name` - by default "#{attribute_name.to_s.singularize.camelcase}Form", but can be a model or other ModelForm
* `reject_if` - 
* `allow_destroy` - 
* TODO more

* `valid?`
* `invalid?`
* `save`
* `save!`
* `model`
* `params`
* `errors` - TODO think how it should be: nested or plain!?

Pending callbacks:

* `before_save`
* `after_save`
* `after_commit`
