# SQLite Triggers

This example demonstrates the use of triggers on the following database:
* book(book_id, book_name, price, quantity, pub_id)
* publisher(pub_id, pub_name)
* book_log(book_id, operation, old_quantity, new_quantity, old_price, new_price)

The data set is in [the script file](book.txt) and the code examples can be run on https://sqliteonline.com/

* Use AFTER UPDATE to log change history.
```sql
CREATE TRIGGER trigger_log_after_update
AFTER UPDATE on book
WHEN OLD.quantity <> NEW.quantity
OR OLD.price <> NEW.price
BEGIN
  INSERT INTO book_log(book_id, operation, old_quantity,
    new_quantity, old_price, new_price)
  VALUES(OLD.book_id, 'UPDATE '||date('now'), OLD.quantity,
    NEW.quantity, OLD.price, NEW.price);
END;

UPDATE book SET quantity = 20 WHERE book_id = 1;

UPDATE book SET price = 100 WHERE book_id = 1;

SELECT * FROM book_log;
```

* Use AFTER DELETE to log change history.
```sql
CREATE TRIGGER trigger_log_after_delete
AFTER DELETE on book
BEGIN
  INSERT into book_log(book_id, operation, old_quantity, old_price)
  VALUES(OLD.book_id, 'DELETE '||date('now'), OLD.quantity, OLD.price);
END;

DELETE FROM book WHERE book_id = 1;

SELECT * FROM book_log;
```

* Use BEFORE UPDATE to validate data.
```sql
CREATE TRIGGER trigger_validate_book_before_update
BEFORE UPDATE ON book
BEGIN
  SELECT CASE
    WHEN NEW.quantity < 0 THEN
    RAISE(ABORT, 'Invalid quantity')
  END;
  SELECT CASE
    WHEN NEW.price <= 0 THEN
    RAISE(ABORT, 'Invalid price')
  END;
END;

UPDATE book SET price = -3 WHERE book_id = 2;
UPDATE book SET quantity = -10 WHERE book_id = 2;
```

* Use AFTER INSERT to trim (`LTRIM()`) leading spaces in publisher name and convert it to upper case by `UPPER()` function.

```sql
CREATE TRIGGER trigger_reformat_publisher_name
AFTER INSERT ON publisher
BEGIN
  UPDATE publisher
  SET pub_name = UPPER(LTRIM(pub_name))
  WHERE pub_id = NEW.pub_id;
END;

INSERT INTO publisher VALUES (3, 'George Routledge and Sons');

SELECT * FROM publisher;
```

* Use BEFORE DELETE to check foreign key constraints.
```sql
CREATE TRIGGER trigger_check_foreign_key_before_delete
BEFORE DELETE on publisher
BEGIN
  SELECT CASE
    WHEN
      (SELECT count(pub_id) FROM book WHERE pub_id = OLD.pub_id) > 0
    THEN
      RAISE(ABORT, "Foreign key constraint violated")
  END;
END;

DELETE FROM publisher WHERE pub_id = 1;
```

* Use AFTER INSERT to add non-existent publishers.
```sql
CREATE TRIGGER trigger_add_null_publisher
AFTER INSERT ON book
WHEN NOT EXISTS(SELECT * FROM publisher WHERE pub_id = NEW.pub_id)
BEGIN
  INSERT INTO publisher(pub_id, pub_name) VALUES (NEW.pub_id, NULL);
END;

INSERT INTO book VALUES (4, 'The Imitation of Christ', 3.99, 55, 4);

SELECT * FROM publisher;
```

References:
* https://www.w3resource.com/sqlite/sqlite-triggers.php
* https://www.sqlitetutorial.net/sqlite-trigger/
* https://www.tutlane.com/tutorial/sqlite/sqlite-triggers
