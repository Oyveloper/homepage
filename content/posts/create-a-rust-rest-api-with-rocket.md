---
title: "Create a Rust Rest Api With Rocket"
date: 2023-01-10T19:25:34+01:00
draft: false
tags: ["Rust", "Rocket", "REST", "API", "Guide"]
---

This article will be going over how to make a very simple REST API with Rust and the Rocket library. Since Rust is more of a low level language you would seldom use it for such a simple example, but it fits the purpouse of focusing on just the web part of this.

## Project setup

First of all we need to setup a project with cargo:

```bash
cargo new simple-rest --bin
cd simple-rest
```

Now we need some dpendencies, namely Rocket. Open up your Cargo.toml file and add the following dependency:

```toml
[dependencies]
rocket = { version = "0.5.0-rc.1", features = ["json"]}
```

Note that we add the json feature. This is important for us to be able to use the automatic JSON serialization later on, and is not easy to figure out from the docs.

Hello world
Next up we'll create our first "Hello World" endpoint. In the main.rs file add the following:

```rs
#[macro_use]
extern crate rocket;

#[rocket::get("/")]
fn index() -> &'static str {
  "Hello, world!"
}

#[rocket::main]
async fn main() {
  rocket::build()
    .mount("/", routes![index])
    .launch()
    .await;
}
```

Let's go over what we just wrote. First we define a GET-endpoint with #[rocket::get("/")]. This is the way we tell rocket to point the "/" endpoint to whatever function we put below. Our function simply returns the string "Hello world", but we can return basically any type we want. We can also the same for POST, UPDATE and all the other HTTP methods.

After that we define the starting point for rocket with the #[rocket::main]. In the main method we build rocket and register our routes.

JSON
In a real REST API you would probably want to return data as JSON instead of just string messages. Luckily, Rocket makes this easy for us with the JSON return type. Every struct that implements Serialize from serde can be fed into JSON and automatically parsed. Lets add a simple user model and return it in an endpoint.

```rs
use rocket::serde::Serialize;

#[derive(Serialize)]
pub struct User {
  pub name: String,
      pub user_id: String,
}
```

Perfect! Now that we have this very simple model we can use it to return some more realistic data on our API.

```rs
use rocket::serde::json::Json;

#[rocket::get("/users")
fn users() -> Json<Vec<User>> {
  // This would be replaced by your logic
  // for retrieving users
  let uers = vec![
    User {
name: String::from("Some Name"),
        user_id: String::from("12234"),
    }
  ];

  // This is where the JSON magic happens
  Json(users)
}
```

The most important things to note above is the Json<Vec<User>> return type, and the return of Json(users). The Json method will serialize the object into JSON and return it as a Responder which all Rocket endpoints must implement. Responer includes fields as status and content, and is used to represent a response from the API. Luckily the Json Responder is ready to use for us. This is the reason we needed the features = ["json"] in our dependency earlier.

We also need to add this endpoint to our app. To do this we add it in the main method:

```rs
#[rocket::main]
async fn main() {
  rocket::build()
    .mount("/", routes![index, users])
    .attach(cors)
    .launch()
    .await;
}
```

## CORS

You probably want to use your API somewhere, and my guess is that you would like to do that from a Javascript application running localy during development. To do this our endpoint must send a CORS (Cross Origin Resource Sharing) header to tell our browser that it is ok for our website to fetch this api, even though they will be on different domains (typically different ports on your local machine during development).

The simplest way to achieve this is with rocket_cors. First we add the dependency to Cargo.toml

```toml
[dependencies]
// Existing dependencies …
rocket_cors = { git = "https://github.com/lawliet89/rocket_cors", branch = "master" }
```

And now we modify our endpoint function to use the cors library:

```rs
#[rocket::main]
async fn main() {
  let allowed_origins = AllowedOrigins::some_exact(&["http://localhost:3000"]);

  // You can also deserialize this
  let cors = rocket_cors::CorsOptions {
    allowed_origins,
      allowed_methods: AllowedMethods::default(),
      allowed_headers: AllowedHeaders::some(&["Authorization", "Accept"]),
      allow_credentials: true,
      ..Default::default()
  }
  .to_cors().expect("Failed to create CORS");


  rocket::build()
    .mount("/", routes![index, users])
    .attach(cors)
    .launch()
    .await;

}
```

That's it! You now have a running REST API that serves a custom data model as JSON that can be accessed from a javascript application!
