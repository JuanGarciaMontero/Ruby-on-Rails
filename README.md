Instalación en local:

Para instalar Rails en Windows, primero deberá instalar Ruby Installer .

Para conocer más métodos de instalación para la mayoría de los sistemas operativos, consulte ruby-lang.org .

ruby --version
sqlite3 --version

gem install rails

rails --version

rails new blog

cd blog

Inicio servidor web: bin/rails server

http://localhost:3000

**************************************************************

Di "Hola", Rails

Comencemos agregando una ruta a nuestro archivo de rutas, config/routes.rben la parte superior del Rails.application.routes.drawbloque:

Rails.application.routes.draw do
  get "/articles", to: "articles#index"

  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
end

La ruta anterior declara que GET /articleslas solicitudes se asignan a la index acción de ArticlesController.

Para crear ArticlesControllery su indexacción, ejecutaremos el generador de controladores (con la --skip-routesopción porque ya tenemos una ruta adecuada):

bin/rails generate controller Articles index --skip-routes

Abramos app/views/articles/index.html.erby reemplacemos su contenido con:

<h1>Hello, Rails!</h1>


http://localhost:3000/articles


**************************************************************

MVC

Generando un modelo

bin/rails generate model Article title:string body:text

Migraciones de bases de datos
(db/migrate/<timestamp>_create_articles.rb) 

class CreateArticles < ActiveRecord::Migration[7.1]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :body

      t.timestamps
    end
  end
end

Ejecutemos nuestra migración con el siguiente comando:

bin/rails db:migrate

Uso de un modelo para interactuar con la base de datos:

$ bin/rails console

En este mensaje, podemos inicializar un nuevo Articleobjeto:

article = Article.new(title: "Hello Rails", body: "I am on Rails!")

article.save

article

Article.find(1)

Article.all

*********************************

Mostrar una lista de artículos

app/controllers/articles_controller.rb

class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end
end

1. El navegador realiza una solicitud: GET http://localhost:3000.
2. Nuestra aplicación Rails recibe esta solicitud.
3. El enrutador Rails asigna la ruta raíz a la indexacción de ArticlesController.
4. La indexacción utiliza el Articlemodelo para recuperar todos los artículos de la base de datos.
5. Rails renderiza automáticamente la app/views/articles/index.html.erbvista.
6. El código ERB en la vista se evalúa para generar HTML.
7. El servidor envía una respuesta que contiene el HTML al navegador.


***********************************************

CRUD

Mostrando un solo artículo

config/routes.rb

Rails.application.routes.draw do
  root "articles#index"

  get "/articles", to: "articles#index"
  get "/articles/:id", to: "articles#show"
end


app/controllers/articles_controller.rb

class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end
end


app/views/articles/show.html.erb

<h1><%= @article.title %></h1>

<p><%= @article.body %></p>


http://localhost:3000/articles/1


**************************************************

Enrutamiento ingenioso:

config/routes.rb

Rails.application.routes.draw do
  root "articles#index"

  resources :articles
end

bin/rails routes

app/views/articles/index.html.erb

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <a href="<%= article_path(article) %>">
        <%= article.title %>
      </a>
    </li>
  <% end %>
</ul>


El link_to asistente representa un vínculo con su primer argumento como el texto del vínculo y su segundo argumento como el destino del vínculo. Si pasamos un objeto modelo como segundo argumento, link_to llamaremos al asistente de ruta apropiado para convertir el objeto en una ruta. Por ejemplo, si pasamos un artículo, link_tolo llamaremos article_path. Entonces app/views/articles/index.html.erb se convierte en:

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <%= link_to article.title, article %>
    </li>
  <% end %>
</ul>

*************************************************************

Crear un nuevo artículo


app/controllers/articles_controller.rbdebajo de la showacción

def new
    @article = Article.new
  end

  def create
    @article = Article.new(title: "...", body: "...")

    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end

Uso de un creador de formularios

app/views/articles/new.html.erb

<h1>New Article</h1>

<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>

Uso de parámetros fuertes

app/controllers/articles_controller.rb

class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render :new, status: :unprocessable_entity
    end
  end

  private
    def article_params
      params.require(:article).permit(:title, :body)
    end
end


Validaciones y visualización de mensajes de error

app/models/article.rb

class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end


app/views/articles/new.html.erb

<h1>New Article</h1>

<%= form_with model: @article do |form| %>
  <div>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
    <% @article.errors.full_messages_for(:title).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.label :body %><br>
    <%= form.text_area :body %><br>
    <% @article.errors.full_messages_for(:body).each do |message| %>
      <div><%= message %></div>
    <% end %>
  </div>

  <div>
    <%= form.submit %>
  </div>
<% end %>


http://localhost:3000/articles/new

Enlacemos a esa página desde la parte inferior de: 

app/views/articles/index.html.erb

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <%= link_to article.title, article %>
    </li>
  <% end %>
</ul>

<%= link_to "New Article", new_article_path %>


Actualización de un artículo


app/controllers/articles_controller.rb

def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render :edit, status: :unprocessable_entity
    end
  end

Uso de parciales para compartir código de vista

Actualicemos app/views/articles/new.html.erb para usar la vía parcial render

<h1>New Article</h1>

<%= render "form", article: @article %>

Y ahora, creemos uno muy similar app/views/articles/edit.html.erb

Terminar

http://localhost:3000/articles/1/edit

app/views/articles/show.html.erb

<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
</ul>


Eliminar un artículo

app/controllers/articles_controller.rb

def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to root_path, status: :see_other
  end


app/views/articles/show.html.erb


<h1><%= @article.title %></h1>

<p><%= @article.body %></p>

<ul>
  <li><%= link_to "Edit", edit_article_path(@article) %></li>
  <li><%= link_to "Destroy", article_path(@article), data: {
                    turbo_method: :delete,
                    turbo_confirm: "Are you sure?"
                  } %></li>
</ul>



Agregar un segundo modelo

Es hora de agregar un segundo modelo a la aplicación. El segundo modelo manejará comentarios sobre artículos.


Generando un modelo

bin/rails generate model Comment commenter:string body:text article:references

Ejecutar migración:
 bin/rails db:migrate


Asociación de modelos

app/models/article.rb

class Article < ApplicationRecord
  has_many :comments

  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end


Agregar una ruta para comentarios

config/routes.rb

Rails.application.routes.draw do
  root "articles#index"

  resources :articles do
    resources :comments
  end
end


Generando un controlador

bin/rails generate controller Comments

app/views/articles/show.html.erb

Añadimos

<h2>Add a comment:</h2>
<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>


Conectemos el create in app/controllers/comments_controller.rb:

class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end


app/views/articles/show.html.erb

<h2>Comments</h2>
<% @article.comments.each do |comment| %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>

  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>


Refactorización

app/views/articles/show.html.erbplantilla. Se está volviendo largo e incómodo. Podemos usar parciales para limpiarlo.


Representación de colecciones parciales

app/views/comments/_comment.html.erb

<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>

<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>


Luego puedes cambiar app/views/articles/show.html.erb

<h2>Comments</h2>
<%= render @article.comments %>

Representación de un formulario parcial

app/views/comments/_form.html.erb

<%= form_with model: [ @article, @article.comments.build ] do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>


app/views/articles/show.html.erb

<h2>Add a comment:</h2>
<%= render 'comments/form' %>


Uso de preocupaciones

bin/rails generate migration AddStatusToArticles status:string
bin/rails generate migration AddStatusToComments status:string


bin/rails db:migrate


app/controllers/articles_controller.rb

  private
    def article_params
      params.require(:article).permit(:title, :body, :status)
    end

app/controllers/comments_controller.rb


  private
    def comment_params
      params.require(:comment).permit(:commenter, :body, :status)
    end

bin/rails db:migrate



class Article < ApplicationRecord
  has_many :comments

  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }

  VALID_STATUSES = ['public', 'private', 'archived']

  validates :status, inclusion: { in: VALID_STATUSES }

  def archived?
    status == 'archived'
  end
end

y en el Commentmodelo:

class Comment < ApplicationRecord
  belongs_to :article

  VALID_STATUSES = ['public', 'private', 'archived']

  validates :status, inclusion: { in: VALID_STATUSES }

  def archived?
    status == 'archived'
  end
end

app/views/articles/index.html.erb

<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <% unless article.archived? %>
      <li>
        <%= link_to article.title, article %>
      </li>
    <% end %>
  <% end %>
</ul>

<%= link_to "New Article", new_article_path %>

app/views/comments/_comment.html.erb

<% unless comment.archived? %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>

  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>

app/models/concerns/visible.rb

module Visible
  def archived?
    status == 'archived'
  end
end

Ahora podemos eliminar la lógica duplicada de cada modelo y en su lugar incluir nuestro nuevo Visiblemódulo:

En app/models/article.rb:

class Article < ApplicationRecord
  include Visible

  has_many :comments

  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end

y en app/models/comment.rb:

class Comment < ApplicationRecord
  include Visible

  belongs_to :article
end

Contador en visible:

module Visible
  extend ActiveSupport::Concern

  VALID_STATUSES = ['public', 'private', 'archived']

  included do
    validates :status, inclusion: { in: VALID_STATUSES }
  end

  class_methods do
    def public_count
      where(status: 'public').count
    end
  end

  def archived?
    status == 'archived'
  end
end


<h1>Articles</h1>

Our blog has <%= Article.public_count %> articles and counting!

<ul>
  <% @articles.each do |article| %>
    <% unless article.archived? %>
      <li>
        <%= link_to article.title, article %>
      </li>
    <% end %>
  <% end %>
</ul>

<%= link_to "New Article", new_article_path %>

app/views/articles/_form.html.erb

<div>
  <%= form.label :status %><br>
  <%= form.select :status, Visible::VALID_STATUSES, selected: article.status || 'public' %>
</div>

y en app/views/comments/_form.html.erb:

<p>
  <%= form.label :status %><br>
  <%= form.select :status, Visible::VALID_STATUSES, selected: 'public' %>
</p>


Eliminar comentarios

app/views/comments/_comment.html.erb

<p>
    <%= link_to "Destroy Comment", [comment.article, comment], data: {
                  turbo_method: :delete,
                  turbo_confirm: "Are you sure?"
                } %>
</p>

app/controllers/comments_controller.rb

def destroy
    @article = Article.find(params[:article_id])
    @comment = @article.comments.find(params[:id])
    @comment.destroy
    redirect_to article_path(@article), status: :see_other
  end


Eliminar objetos asociados

app/models/article.rb

class Article < ApplicationRecord
    include Visible
  
    has_many :comments, dependent: :destroy
  
    validates :title, presence: true
    validates :body, presence: true, length: { minimum: 10 }
end

Seguridad

Autenticación básica
app/controllers/articles_controller.rb

class ArticlesController < ApplicationController
  http_basic_authenticate_with name: "juan", password: "juan", except: [:index, :show]

  def index
    @articles = Article.all
  end

Solo elimina los usuarios autenticados:
app/controllers/comments_controller.rb

class CommentsController < ApplicationController

  http_basic_authenticate_with name: "juan", password: "juan", only: 
:destroy

