# Ghostwriter

Ghostwriter rewrites HTML as plain text while preserving as much legibility and functionality as possible.

It's sort of like a reverse-markdown.

## But Why, Though?

* Spam filters tend to like emails with a plain text alternative
* Some email clients won't or can’t handle HTML at all
* Some people explicitly choose plaintext just by preference or accessibility

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'ghostwriter'
```

And then execute:

    bundle

Or install it manually with:

    gem install ghostwriter

## Usage

Create a `Ghostwriter::Writer` with the html you want modified, and call `#textify`:

```ruby
html = '<html><body><p>This is some markup <a href="tenjin.ca">and a link</a></p><p>Other tags translate, too</p></body></html>'

Ghostwriter::Writer.new(html).textify
```

Produces:

```
This is some markup and a link (tenjin.ca)

Other tags translate, too
```

### Links

Links are converted to the link text followed by the link target in brackets:

```ruby
html = '<html><body>Visit our <a href="https://example.com">Website</a><body></html>'
Ghostwriter::Writer.new(html).textify
```

Produces:

```
Visit our Website (https://example.com)
```

#### Relative Links

Since emails are wholly distinct from your web address, relative links might break.

To avoid this problem, either use the `<base>` header tag:

```ruby
html = <<~HTML
   <html>
      <head>
         <base href="https://www.example.com/">
      </head>
      <body>
         Relative links get <a href="/contact">expanded</a> using the head's base tag. 
      </body>
   </html>
HTML

Ghostwriter::Writer.new(html).textify
```

Produces:

```
Relative links get expanded (https://www.example.com//contact) using the head's base tag.
```

Or you can use the `link_base` parameter:

```ruby
html = '<html><body>Relative links get <a href="/contact">expanded</a></body></html> using the link_base parmeter, too.'

Ghostwriter::Writer.new(html).textify(link_base: 'tenjin.ca')
```

Produces:

```
"Relative links get expanded (tenjin.ca/contact) using the link_base parmeter, too."
```

### Lists

Lists are converted, too. They get padded with newlines and are given simple markers:

```ruby
html = <<~HTML
    <ul>
        <li>Planes</li>
        <li>Trains</li>
        <li>Automobiles</li>
    </ul>
    <ol>
        <li>I get knocked down</li>
        <li>I get up again</li>
        <li>Never gonna keep me down</li>
    </ol>
HTML

Ghostwriter::Writer.new(html).textify
```

Produces:

```

- Planes
- Trains
- Automobiles

1. I get knocked down
2. I get up again
3. Never gonna keep me down

```

### Tables

Tables are often used email structuring because support for more modern CSS is inconsistent.

Ghostwriter tries to maintain table structure, but this will quickly devolve for complex structures.

```ruby
html = <<~HTML
   <html>
      <head>
         <base href="https://www.example.com/">
      </head>
      <body>
         <table>
            <thead>
                <tr>
                    <th>Ship</th>
                    <th>Captain</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Enterprise</td>
                    <td>Jean-Luc Picard</td>
                </tr>
                <tr>
                    <td>TARDIS</td>
                    <td>The Doctor</td>
                </tr>
                <tr>
                    <td>Planet Express Ship</td>
                    <td>Turanga Leela</td>
                </tr>
            </tbody>
         </table> 
      </body>
   </html>
HTML

Ghostwriter::Writer.new(html).textify
```

Produces:

```
| Ship                | Captain         |
|---------------------|-----------------|
| Enterprise          | Jean-Luc Picard |
| TARDIS              | The Doctor      |
| Planet Express Ship | Turanga Leela   |
```

### Mail Gem Example

To use `#textify` with the [mail](https://github.com/mikel/mail) gem, just provide the text-part by pasisng the html
through Ghostwriter:

```ruby
require 'mail'

html = 'My email and a <a href="https://tenjin.ca">link</a>'

Mail.deliver do
   to 'bob@example.com'
   from 'dot@example.com'
   subject 'Using Ghostwriter with Mail'

   html_part do
      content_type 'text/html; charset=UTF-8'
      body html
   end

   text_part do
      body Ghostwriter::Writer.new(html).textify
   end
end

```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/TenjinInc/ghostwriter

This project is intended to be a friendly space for collaboration, and contributors are expected to adhere to the
[Contributor Covenant](contributor-covenant.org) code of conduct.

### Core Developers

After checking out the repo, run `bundle install` to install dependencies. Then, run `rake spec` to run the tests. You
can also run `bin/console` for an interactive prompt that will allow you to experiment.

#### Local Install

To install this gem onto your local machine only, run

`bundle exec rake install`

#### Gem Release

To release a gem to the world at large

1. Update the version number in `version.rb`,
2. Run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push
   the `.gem` file to [rubygems.org](https://rubygems.org).
3. Do a wee dance

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
