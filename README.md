# Ecommerce-django Documentation
## Overview
This is a Django-based shopping app that integrates Stripe for payment processing. The app allows users to browse products, add items to their cart, and securely checkout using Stripe.

## Features
- User Authentication (Login, Signup)
  
- Product Listing & Details
- Shopping Cart
- Stripe Payment Integration
- Order Management
- User Profile & Order History

## Setup and Installation
- Django is installed using pip, with this command:
```
python -m pip install Django

```
- Set up a virtual environment:
```
virtualenv venv
source venv/bin/activate

```
- Create a superuser:
```
python manage.py createsuperuser

```
- Start the development server:
```
python manage.py runserver

```
- Install stripe
```
python -m pip install stripe==9.3.0

```
- Install celery
```
python -m pip install celery==5.4.0
```
- Install flower
```
python -m pip install flower==2.0.1

```
### Prerequisites
- Python 3.x
- Django 5.x or higher
- Stripe API keys (publishable and secret keys)
- Virtual Environment
- flower 2.x
- celery 5.x

### Model
- Product Model
```
class Product(models.Model):
    category = models.ForeignKey(
        Category,
        related_name='products',
        on_delete=models.CASCADE
    )
    name = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200)
    image = models.ImageField(
        upload_to='products/%Y/%m/%d',
        blank=True
    )
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    available = models.BooleanField(default=True)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)

```
- Order Model
```
class Order(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    email = models.EmailField()
    address = models.CharField(max_length=250)
    postal_code = models.CharField(max_length=20)
    city = models.CharField(max_length=100)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    paid = models.BooleanField(default=False)
    stripe_id = models.CharField(max_length=250, blank=True)

```

### Views
Provide documentation for how views are structured, e.g., class-based views or function-based views. Example for product listing:
```
from django.shortcuts import get_object_or_404, render

from cart.forms import CartAddProductForm
from .models import Category, Product


def product_list(request, category_slug=None):
    category = None
    categories = Category.objects.all()
    products = Product.objects.filter(available=True)
    if category_slug:
        category = get_object_or_404(Category, slug=category_slug)
        products = products.filter(category=category)
    return render(
        request,
        'shop/product/list.html',
        {
            'category': category,
            'categories': categories,
            'products': products,
        },
    )


def product_detail(request, id, slug):
    product = get_object_or_404(
        Product, id=id, slug=slug, available=True
    )
    cart_product_form = CartAddProductForm()
    return render(
        request,
        'shop/product/detail.html',
        {'product': product, 'cart_product_form': cart_product_form},
    )

```

### Templates
The templates are stored in the templates/ directory. Here's an example of the product listing template:
```
{% extends "shop/base.html" %}
{% load static %}

{% block title %}
  {% if category %}{{ category.name }}{% else %}Products{% endif %}
{% endblock %}

{% block content %}
  <div id="sidebar">
    <h3>Categories</h3>
    <ul>
      <li {% if not category %}class="selected"{% endif %}>
        <a href="{% url "shop:product_list" %}">All</a>
      </li>
      {% for c in categories %}
        <li {% if category.slug == c.slug %}class="selected"{% endif %}>
          <a href="{{ c.get_absolute_url }}">{{ c.name }}</a>
        </li>
      {% endfor %}
    </ul>
  </div>
  <div id="main" class="product-list">
    <h1>{% if category %}{{ category.name }}{% else %}Products{% endif %}</h1>
    {% for product in products %}
      <div class="item">
        <a href="{{ product.get_absolute_url }}">
          <img src="{% if product.image %}{{ product.image.url }}{% else %}{% static "img/no_image.png" %}{% endif %}">
        </a>
        <a href="{{ product.get_absolute_url }}">{{ product.name }}</a>
        <br>
        ${{ product.price }}
      </div>
    {% endfor %}
  </div>
{% endblock %}

```
### URLs
Document the URL structure:
```
from django.urls import path

from . import views

app_name = 'shop'

urlpatterns = [
    path('', views.product_list, name='product_list'),
    path(
        '<slug:category_slug>/',
        views.product_list,
        name='product_list_by_category',
    ),
    path(
        '<int:id>/<slug:slug>/',
        views.product_detail,
        name='product_detail',
    ),
]

```
### Admin Panel
The Django admin is used to manage products, categories, orders, and users. You can access the admin panel at /admin/. Make sure to register your models in admin.py:
```
from django.contrib import admin

from .models import Category, Product


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = [
        'name',
        'slug',
        'price',
        'available',
        'created',
        'updated',
    ]
    list_filter = ['available', 'created', 'updated']
    list_editable = ['price', 'available']
    prepopulated_fields = {'slug': ('name',)}
```
