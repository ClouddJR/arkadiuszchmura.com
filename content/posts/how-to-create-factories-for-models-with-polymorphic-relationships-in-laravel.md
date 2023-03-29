---
title: How to create factories for models with polymorphic relationships in Laravel
date: 2023-03-29
summary: Factories for these models are a bit trickier to implement than the standard ones.
---

## Introduction

In this post, I will show you how to create factories for models that have polymorphic relationships with other models in your app.

This post assumes that you are familiar with polymorphic relationships. If that's not true, head to [the documentation](https://laravel.com/docs/10.x/eloquent-relationships#polymorphic-relationships) to learn more.

## Solution

As an example, let's consider an app where we allow users to rate games or movies. For that, we define two Eloquent models - `Game` and `Movie`. Additionally, to avoid repetition, we use a single `Rating` model for both entities.

A simplified database structure for our app could look like this:

```php
games
    id - integer
    name - string
 
movies
    id - integer
    name - string
 
ratings
    id - integer
    rating - integer
    rateable_id - integer
    rateable_type - string
```

Now, let's move on to factories. For both `Game` and `Movie`, it would be a simple, regular factory. Something like this:

```php
class GameFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->word(),
        ];
    }
}
```

I'm only showing the code for the `GameFactory`, as the `MovieFactory` would look almost identical.

The factory for our `Rating` model is going to be a bit more complicated, and we could define it like this:

```php
class RatingFactory extends Factory
{
    public function definition(): array
    {
        return [
            'rateable_id' => Game::factory(),
            'rateable_type' => function (array $attributes) {
                return Game::find($attributes['rateable_id'])->getMorphClass();
            }
        ];
    }
}
```

It's a typical pattern to assign a new factory instance to the foreign key of the relationship (hence the `'rateable_id' => Game::factory()`). Thanks to this, when we create a new `Rating` using the factory, a new `Game` instance will be automatically created for us.

The `rateable_type` attribute is more interesting. Potentially, we could hardcode the value to `App\Models\Game`, and it would work just fine because, by default, Laravel uses the fully qualified class name to store the "type" of the related model. However, I like to decouple my app internals from the database and usually change these values.

To do that, you can use the following code (placed in the `boot` method of one of your service providers) to store simple strings as the "type" in the database:

```php
Relation::enforceMorphMap([
    'game' => 'App\Models\Game',
    'movie' => 'App\Models\Movie',
]);
```

But after that change, we must provide correct values to our `RatingFactory`. That's precisely why we we used the `getMorphClass()` on the model instance. It allows us to get the valid morph alias at runtime. 

Also, we used a closure to access other attributes defined in the factory (in particular, the `rateable_id`) to find the newly created `Game` model on which we call the `getMorphClass()` method.

To use the factory and create a new `Rating`, we can now call `Rating::factory()->create()`.

Notice that when we use this factory, by default, it will associate the new `Rating` model with a new `Game`. What if we would like to use a `Movie` model instead (or any other that might be added in the future)? 

To do that, we could leverage [factory states](https://laravel.com/docs/10.x/eloquent-factories#factory-states). We can define a method called `forMovie` in our factory that will only modify the `rateable_id` and `rateable_type` attributes accordingly (leaving the remaining ones untouched):

```php
public function forMovie(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'rateable_id' => Movie::factory(),
            'rateable_type' => function (array $attributes) {
                return Movie::find($attributes['rateable_id'])->getMorphClass();
            }
        ];
    });
}
```

Then, you can create a new `Rating` associated with a `Movie` by calling `Rating::factory()->forMovie()->create()`.

Similarly, you could define a separate method for any other model related to `Rating` that you might add in the future.

## Summary

In summary, creating factories for models with polymorphic relationships in Laravel can be a bit trickier than the standard ones. However, by following the steps outlined in this post, I hope you won't have any problems with that.

If you have any questions, feel free to leave a comment here or reach out to me on [Twitter](https://twitter.com/ClouddJR/).

