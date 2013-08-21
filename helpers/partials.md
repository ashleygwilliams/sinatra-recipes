# Implementation of Rails style partials

Using partials in your views is a great way to keep them clean.  Since Sinatra takes the hands off approach to framework design, you'll have to implement a partial handler yourself.

Depending on the complexity of partials you'd like to use, you should pick between one of the following three strategies:

- Simple Partial
- Intermediate Partial (underscores and local variables)
- Advanced Partial (local variables)

## Simple Partial
The simplest implenetation of a partial create a method that takes a single parameter, `template` and uses the same mechanism that is used to render a view, i.e. `haml` (can be replaced with `erb`). However, the layout option is set to false so that the layout is not rendered again.

```ruby
# Usage: partial :foo
helpers do
  def partial(template)
    haml template, :layout => false
  end
end
```

Using the code above, calling partial(:header) will render the view `header.haml` in your views folder.

## Intermediate Partial
The simple implementation is fine, but it simply renders a view within a view; nothing to sophisticated. Web development is just fancy string concatenation after all :)

Usually we want to do more than simply render a view within a view though, so let's step it up a notch.

### _underscore Naming Convention
First off: Rails uses a convention where partials are named beginning with an underscore, but can be referenced by using a symbol with the template name sans underscore.

e.g. `partial :header` renders `_header.haml`

In order to do this we can modify our partial to include the line `template=('_' + template.to_s).to_sym`. This gives us:

```ruby
helpers do
    
  def partial(template)
    template=('_' + template.to_s).to_sym
    haml template, :layout => false      
  end
    
end
```  

### Passing Local Variables
Secondly: it is also useful to be able to pass local variables to your partial. To implement this we can add a `locals` parameter to our partial. This parameter will hold a hash, and will default to `nil`. We will pass this `locals` parameter to `haml` alongside our template, like so: `haml(template,{:layout => false}, locals)`

Rails takes this a step further: if the name of the local variable is the same as the name of the partial, you do not need to explicitly set it. This is handled by the line: `locals = locals.is_a?(Hash) ? locals : {template.to_sym => locals}`. This checks to see if locals is a hash, and if it is not, it will build a hash with a single key value pair, where the key is the name of the template and the value is the non-hash value of `locals`.

This leaves us with our finished intermediate partial, which now supprots the underscore naming convention and the explicit and implicit passing of local variables:

```ruby
helpers do
    
  def int_partial(template,locals=nil)
    locals = locals.is_a?(Hash) ? locals : {template.to_sym =>         locals}
    template=('_' + template.to_s).to_sym
    erb(template,{:layout => false},locals)      
  end
    
end
```


## Advanced Partial

A more advanced version that would handle passing local options, and looping over a hash would look like:

```ruby
# Render the page once:
# Usage: partial :foo
#
# foo will be rendered once for each element in the array, passing in a local
# variable named "foo"
# Usage: partial :foo, :collection => @my_foos

helpers do
  def partial(template, *args)
    options = args.extract_options!
    options.merge!(:layout => false)
    if collection = options.delete(:collection) then
      collection.inject([]) do |buffer, member|
        buffer << haml(template, options.merge(
                                  :layout => false, 
                                  :locals => {template.to_sym => member}
                                )
                     )
      end.join("\n")
    else
      haml(template, options)
    end
  end
end
```
