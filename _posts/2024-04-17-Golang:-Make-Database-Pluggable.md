
Most startups aim to quickly build a POC and validate their ideas before fully committing. In the early stages, they often rely on a single database for all their needs. However, as the startup scales, this approach can become a bottleneck. A single database might not be able to handle every use case effectively, leading to the need for multiple databases. Sometimes, it might even be necessary to migrate certain tables from one database to another.

Therefore, it is wise to invest a little effort upfront to abstract database and queue-related methods. This makes it easier to swap them out in the future without changing the application or domain layer.

Let's write some code!

### Step 1: 
Start by defining interfaces for database operations

```golang
package db

type UserRepository interface {
    CreateUser(user *User) error
    GetUserByID(id string) (*User, error)
    UpdateUser(user *User) error
    DeleteUser(id string) error
}

type ProductRepository interface {
    CreateProduct(product *Product) error
    GetProductByID(id string) (*Product, error)
    UpdateProduct(product *Product) error
    DeleteProduct(id string) error
}

type User struct {
    ID   string
    Name string
}

type Product struct {
    ID    string
    Name  string
    Price float64
}

```

### Step 2:
Next, implement these interfaces for the initial database you chose, For instance if you start with PostgresSQL

```golang
package postgres

import (
    "database/sql"
    "yourapp/db"
    _ "github.com/lib/pq"
)

type PostgresUserRepository struct {
    DB *sql.DB
}

func (r *PostgresUserRepository) CreateUser(user *db.User) error {
    _, err := r.DB.Exec("INSERT INTO users (id, name) VALUES ($1, $2)", user.ID, user.Name)
    return err
}

func (r *PostgresUserRepository) GetUserByID(id string) (*db.User, error) {
    row := r.DB.QueryRow("SELECT id, name FROM users WHERE id = $1", id)
    user := &db.User{}
    err := row.Scan(&user.ID, &user.Name)
    if err != nil {
        return nil, err
    }
    return user, nil
}

func (r *PostgresUserRepository) UpdateUser(user *db.User) error {
    _, err := r.DB.Exec("UPDATE users SET name = $1 WHERE id = $2", user.Name, user.ID)
    return err
}

func (r *PostgresUserRepository) DeleteUser(id string) error {
    _, err := r.DB.Exec("DELETE FROM users WHERE id = $1", id)
    return err
}
```

### Step 3: Use dependency injection

Inject these implementations into your application layer, ensuring your application code interacts with the interfaces rather than the concrete implementations.

```golang
package main

import (
    "database/sql"
    "yourapp/db"
    "yourapp/postgres"
    _ "github.com/lib/pq"
)

func main() {
    dbConn, err := sql.Open("postgres", "your connection string")
    if err != nil {
        panic(err)
    }

    userRepo := &postgres.PostgresUserRepository{DB: dbConn}

    // Now you can use userRepo to interact with users
    user := &db.User{ID: "1", Name: "John Doe"}
    err = userRepo.CreateUser(user)
    if err != nil {
        panic(err)
    }

    fetchedUser, err := userRepo.GetUserByID("1")
    if err != nil {
        panic(err)
    }
    fmt.Println(fetchedUser)
}
```

### Step 4: Switching implementation:


When it’s time to switch databases or add another one, implement the same interfaces for the new database. For example, if you move to MongoDB:

```golang
package mongo

import (
    "context"
    "yourapp/db"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

type MongoUserRepository struct {
    Collection *mongo.Collection
}

func (r *MongoUserRepository) CreateUser(user *db.User) error {
    _, err := r.Collection.InsertOne(context.TODO(), user)
    return err
}

func (r *MongoUserRepository) GetUserByID(id string) (*db.User, error) {
    result := r.Collection.FindOne(context.TODO(), bson.M{"id": id})
    user := &db.User{}
    err := result.Decode(user)
    if err != nil {
        return nil, err
    }
    return user, nil
}

func (r *MongoUserRepository) UpdateUser(user *db.User) error {
    _, err := r.Collection.UpdateOne(context.TODO(), bson.M{"id": user.ID}, bson.M{"$set": user})
    return err
}

func (r *MongoUserRepository) DeleteUser(id string) error {
    _, err := r.Collection.DeleteOne(context.TODO(), bson.M{"id": id})
    return err
}
```

And update your main application to use the new implementation:

```golang
package main

import (
    "context"
    "yourapp/db"
    "yourapp/mongo"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

func main() {
    client, err := mongo.Connect(context.TODO(), options.Client().ApplyURI("your connection string"))
    if err != nil {
        panic(err)
    }

    userRepo := &mongo.MongoUserRepository{Collection: client.Database("yourdb").Collection("users")}

    // Now you can use userRepo to interact with users
    user := &db.User{ID: "1", Name: "John Doe"}
    err = userRepo.CreateUser(user)
    if err != nil {
        panic(err)
    }

    fetchedUser, err := userRepo.GetUserByID("1")
    if err != nil {
        panic(err)
    }
    fmt.Println(fetchedUser)
}

```

### Conclusion

By investing a little effort upfront to abstract your database and queue-related methods using Golang’s design patterns, you can ensure your startup’s architecture remains flexible and scalable. This abstraction allows you to swap out implementations as your needs evolve, without significant changes to your application or domain layer.

This future-proof approach can save time and effort in the long run, allowing your startup to adapt to new challenges and opportunities with ease.

Happy coding!
