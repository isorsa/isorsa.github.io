---
title: Translating your website with Google Translate?
published_at: 2022-03-23 12:13:14
updated_at: 2022-03-23 12:13:14
---

<p>
Recently, someone tried to translate our website with Google translate while filling a form. That caused a 500 error.
Here is the page with navigation in english and spannish translated with Google Translate:
</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-03-23/navigation_eng.png" alt="navigation links in english" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-03-23/navigation_spa.png" alt="navigation links in spannish" />
  </div>
</div>

<br />
<p>And, here is the code that renders and handles the navigation:</p>

```
  # View - index.html.erb
  <%= form_with url: "/books", method: :get do |form| %>
    <ul class="breadcrumb">
      <% @breadcrumbs.each do |name| %>
        <li><%= form.submit name, name: "navigation", class: 'btn-link' %></li>
      <% end %>
    </ul>
  <% end %>

  # Controller - could be any action
  def index
    @books = Book.all
    @breadcrumbs = ['Page 1', 'Page 2', 'Page 3']
    if params['navigation']
      # do something with params['navigation'], for ex. fetch records, load associations etc...
    end
  end
```

<br />
<p>
The code was working for many years without any issues until someone translated the website with GT. We assumed that because we defined breadcrumb values we do not need to validate the value of the navigation parameter. Here is the log output:
</p>

<div class="flex-row">
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-03-23/navigation_log_eng.png" alt="navigation links in english" />
  </div>
  <div class="flex-column">
    <img class="img-responsive img-thumbnail" width="100%" src="img/services/2022-03-23/navigation_log_spa.png" alt="navigation links in spannish" />
  </div>
</div>

<br />

<p>It could be fixed with something simple as:</p>

```
  if @breadcrumbs.includes?(params['navigation'])
    # do something with params['navigation'], for ex. fetch records, load associations etc...
  else
    # display an error or redirect back
  end
```
