# roar-rails

_Makes using Roar's representers in your Rails app fun._

Roar is a framework for parsing and rendering REST documents. For a better overview about representers please check the [roar repository](https://github.com/apotonick/roar#roar).

## Features

* Rendering with responders
* Parsing incoming documents
* URL helpers in representers
* Better tests
* Autoloading

This gem works with all Rails >= 3.x.

## Rendering with #respond_with

roar-rails provides a number of baked-in rendering methods.

### Conventional Rendering

Easily render resources using representers with the built-in responder.

```ruby
class SingersController < ApplicationController
  include Roar::Rails::ControllerAdditions
  respond_to :json

  def show
    singer = Singer.find_by_id(params[:id])
    respond_with singer
  end
end
```

The representer name will be infered from the passed model class (e.g. a `Singer` instance gets the `SingerRepresenter`). If the passed model is a collection it will be extended using a representer. The representer name will be computed from the controller name (e.g. a `SingersController` uses the `SingersRepresenter`).

Need to use a representer with a different name than your model? You may always pass it in using the `:represent_with` option:

```ruby
respond_with singers, :represent_with => MusicianCollectionRepresenter
end
```

### Represents Configuration

If you don't want to use conventions or pass representers you can configure them on the class level using `::represents`. This will also call `respond_to` for you.

```ruby
class SingersController < ApplicationController
  represents :json, Musician
```
This will use the `MusicianRepresenter` for models and `MusiciansRepresenter` for representing collections.

Note that `::represents` also allows fine-tuning.

```ruby
class SingersController < ApplicationController
  represents :json, :entity => MusicianRepresenter, :collection => MusicianCollectionRepresenter
```

You might pass strings as representer names to `::represents`, they will be constantized at run-time when needed.


### Old API Support

If you don't want to write a dedicated representer for a collection of items (highly recommended, thou) but rather use a representer for each item, use the `:represent_items_with` option.

```ruby
class SingersController < ApplicationController

  def index
    singers = Musician.find(:all)
    respond_with singers, :represent_items_with => SingerRepresenter
  end
end
```


## Parsing incoming documents

In `#create` and `#update` actions it is often necessary to parse the incoming representation and map it to a model instance. Use the `#consume!` method for this.

```ruby
class SingersController < ApplicationController
  respond_to :json

  def create
    singer = Singer.new
    consume!(singer)

    respond_with singer
  end
end
```

The `consume!` call will roughly do the following.

```ruby
singer.
  extend(SingerRepresenter)
  from_json(request.body)
```

So, `#consume!` helps you figuring out the representer module and reading the incoming document.

Note that it respects settings from `#represents`. It uses the same mechanics known from `#respond_with` to choose a representer.

```ruby
consume!(singer, :represent_with => MusicianRepresenter)
```

## Using Decorators

If you prefer roar's decorator approach over extend, just go for it. roar-rails will figure out automatically which represent strategy to use. Be sure to use roar >= 0.11.17.

```ruby
class SingerRepresenter < Roar::Decorator
  include Roar::Representer::JSON

  property :name

  link :self do
    singer_url(represented)
  end
end
```

In decorators' link blocks you currently have to use `represented` to get the actual represented model (this is `self` in module representers).

## Passing Options

Both rendering and consuming support passing user options to the representer.

With `#respond_with`, any additional options will be passed to `to_json` (or whatever format you're using).

```ruby
respond_with @singer, :current_user => current_user
```

Same goes with `#consume!`, passing options to `from_json`.

```ruby
consume! Singer.new, :current_user => current_user
```


## URL Helpers

Any URL helpers from the Rails app are automatically available in representers.

```ruby
module FruitRepresenter
  include Roar::Representer::JSON
  include Roar::Representer::Feature::Hypermedia

  link :self do
    fruit_url self
  end
end
```
To get the hyperlinks up and running, please make sure to set the right _host name_ in your environment files (config/environments):

```ruby
config.representer.default_url_options = {:host => "127.0.0.1:3000"}
```

Attention: If you are using representers from a gem your Rails URL helpers might not work in these modules. This is due to a [loading order problem](https://groups.google.com/forum/?fromgroups#!topic/rubyonrails-core/5tG5unZ8jDQ) in Rails. As a workaround, don't require the representers in the gem but load them as lately as possible, usually it works when you require in the controller. We are working on fixing that problem.

## Testing

## Autoloading

Put your representers in `app/representers` and they will be autoloaded by Rails. Also, frequently used modules as media representers and features don't need to be required manually.


## Contributors

* [Railslove](http://www.railslove.de) and especially Michael Bumann [bumi] have heavily supported development of roar-rails ("resource :singers").

[responders]: https://github.com/plataformatec/responders

## License

Roar-rails is released under the [MIT License](http://www.opensource.org/licenses/MIT).