# 1 .3.3 Model-View-Controller (MVC)

- `app/controllers/application_controller.rb`を編集<br>

```rb:application_controller.rb
class ApplicationController < ActionController::Base
  def hello
    render html: 'hello, world!'
  end
end
```

- `config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  root 'application#hello'
end
```

+ `http://localhost:3000/`にアクセスしてみる<br>

