# SQL_PG_CS

```SQL
-- ENUMish role
-- Timestemp for current date
-- טבלת משתמשים (Users)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) CHECK (role IN ('customer', 'restaurant_owner')) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Basic forign key example
-- ON DELETE CASCADE (delete me if my forign key does not exist in my original table)
-- טבלת מסעדות (Restaurants)
CREATE TABLE restaurants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    owner_id INT NOT NULL,
    location VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE CASCADE
);

-- CHECK(condition using the field) will constraint the parameter that can recive
-- DECIMAL (10,2) will take float rounder to 2 digit after the .
--      so basicly 10 didgit at most that followed by 2 digits at most
-- טבלת מנות (Dishes)
CREATE TABLE dishes (
    id SERIAL PRIMARY KEY,
    restaurant_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) CHECK(price > 0) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id) ON DELETE CASCADE
);

-- some intresting way to create the other part of my ordering app can look at syntax from here
-- טבלת הזמנות (Orders)
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    restaurant_id INT NOT NULL,
    status VARCHAR(50) CHECK(status IN ('pending', 'preparing', 'delivered', 'cancelled')) NOT NULL DEFAULT 'pending',
    total_price DECIMAL(10,2) CHECK(total_price >= 0) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id) ON DELETE CASCADE
);

-- טבלת פרטי הזמנה (Order_Items)
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    dish_id INT NOT NULL,
    quantity INT CHECK(quantity > 0) NOT NULL,
    price DECIMAL(10,2) CHECK(price > 0) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (dish_id) REFERENCES dishes(id) ON DELETE CASCADE
);

-- טבלת תשלומים (Payments)
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    order_id INT NOT NULL UNIQUE,
    amount DECIMAL(10,2) CHECK(amount > 0) NOT NULL,
    payment_method VARCHAR(50) CHECK(payment_method IN ('credit_card', 'paypal', 'cash')) NOT NULL,
    payment_status VARCHAR(50) CHECK(payment_status IN ('pending', 'completed', 'failed')) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE
);



-- OTHER SYNTAX AND FUNCTION EXAMPLES


-- טבלת משתמשים (Users)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) CHECK (role IN ('librarian', 'reader')) NOT NULL,
    account_minus NUMERIC DEFAULT 0 CHECK(account_minus >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- UNIQUE here is to make sure there is only one line with the pair of author and title...
-- cant be that user added the same book twice like this.
-- טבלת ספרים (Books)
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(100) NOT NULL,
    description TEXT,
    release_date DATE NOT NULL,
    num_of_available_copies INT DEFAULT 1 CHECK(num_of_available_copies > 0),
    UNIQUE(title, author)
);

-- טבלת קטגוריות (Categories)
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    description TEXT
);

-- I can set the primary key to by a unique couple as well
-- טבלה מקשרת בין ספרים לקטגוריות (Categorized_Books)
CREATE TABLE categorized_books (
    book_id INT NOT NULL,
    category_id INT NOT NULL,
    PRIMARY KEY (book_id, category_id),
    FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE
);

-- טבלת השאלות (Borrowed_Books)
CREATE TABLE borrowed_books (
    id SERIAL PRIMARY KEY,
    book_id INT NOT NULL,
    user_id INT NOT NULL,
    taken_date DATE NOT NULL,
    return_expected_date DATE NOT NULL CHECK (return_expected_date >= taken_date),
    FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);


-- this is an exmaple of how to trigger a function to validate on every update or insert that
-- I didn't let user borrow more then 3 books...
-- can also do it in code but this has its uses for better performance sometimes and also make sure i dont make a mistake later on.

-- טריגר למניעת השאלת יותר מ-3 ספרים במקביל
CREATE OR REPLACE FUNCTION check_borrow_limit() RETURNS TRIGGER AS $$
BEGIN
    IF (SELECT COUNT(*) FROM borrowed_books WHERE user_id = NEW.user_id) >= 3 THEN
        RAISE EXCEPTION 'User cannot borrow more than 3 books at a time';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER enforce_borrow_limit
BEFORE INSERT ON borrowed_books
FOR EACH ROW EXECUTE FUNCTION check_borrow_limit();





```
