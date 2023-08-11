# Isolations & locks in Percona and PostgresSql

Investigation different common isolation problems for Percona (MySql fork) and PostgreSql on different isolation levels
and hw to solve them

Problems:

- Lost Updates
- Dirty Read
- Non-repeatable Reads
- Phantom Reads

Marks:

✅ - solving on current level

❌ - not solving on current level

## Percona

|Isolation Levels  | Lost Updates | Dirty Read | Non-repeatable Reads | Phantom Reads |
|------------------|---|------------------------------------------|---------------------|---------------|
| Read Uncommitted | ✅ | ❌ | ❌ | ❌ | 
| Read Committed   | ✅ | ✅ | ❌ | ❌ | 
| Repeatable Read  | ✅ | ✅ | ✅ | ❌ | 
| Serializable     | ✅ | ✅ | ✅ | ✅ | 

**Examples**:
1. `Lost Updates`:

TRX1:
`BEGIN; UPDATE users SET username = 'Bob' WHERE id = 1;`

TRX2:
`BEGIN; UPDATE users SET username = 'Robert' WHERE id = 1;`

TRX1: `SELECT username FROM users WHERE id = 1; -- we will see Robert`

2. `Dirty Read`:

TRX1:
`BEGIN; SELECT email FROM users WHERE id = 1;`

TRX2: 
`BEGIN; UPDATE users SET  email = 'updated@mail.com' WHERE id = 1;`

TRX1:
`SELECT email FROM users WHERE id = 1; -- we will see updated@mail.com`

3. `Non-repeatable Reads`:

TRX1:
`BEGIN; SELECT email FROM users WHERE id = 1;`

TRX2:
`BEGIN; UPDATE users SET  email = 'nonrepeateble@mail.com' WHERE id = 1; COMMIT;`

TRX1:
`SELECT email FROM users WHERE id = 1; -- we will see nonrepeateble@mail.com`

4. `Phantom Reads`:

TRX1:
`BEGIN; SELECT email FROM users WHERE birth_date > '2000-01-01';`

TRX2:
`BEGIN; INSERT INTO users VALUES (null, 'new_user', 'phanto-reads@gmail.com', '2001-01-01'); COMMIT;`

TRX1:
`SELECT email FROM users WHERE birth_date > '2000-01-01'; -- we will see new row in range of results`

## PostgreSql

| Isolation Levels | Lost Updates | Dirty Read | Non-repeatable Reads | Phantom Reads |
|------------------|---|-----------------------------------------|---------------------|---|
| Read Committed   | ✅ | ✅ | ❌ | ❌ | 
| Repeatable Read  | ✅ | ✅ | ✅ | ✅ | 
| Serializable     | ✅ | ✅ | ✅ | ✅ | 

In PostgreSQL, you can request any of the four standard transaction isolation levels, but internally only three distinct isolation levels are implemented, i.e., PostgreSQL's Read Uncommitted mode behaves like Read Committed. This is because it is the only sensible way to map the standard isolation levels to PostgreSQL's multiversion concurrency control architecture.

Repeatable Read implementation does not allow phantom reads. This is acceptable under the SQL standard because the standard specifies which anomalies must not occur at certain isolation levels

**Examples**:
1. `Lost Updates`:

TRX1:
`BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; UPDATE users SET username = 'Bob' WHERE id = 1;`

TRX2:
`BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ; UPDATE users SET username = 'Robert' WHERE id = 1;`

TRX1: `SELECT username FROM users WHERE id = 1; -- we will see Robert`

2. `Dirty Read`:

TRX1:
`BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; SELECT email FROM users WHERE id = 1;`

TRX2:
`BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; UPDATE users SET  email = 'updated@mail.com' WHERE id = 1;`

TRX1:
`SELECT email FROM users WHERE id = 1; -- we will see updated@mail.com`

3. `Non-repeatable Reads`:

TRX1:
`BEGIN; SELECT email FROM users WHERE id = 1;`

TRX2:
`BEGIN; UPDATE users SET  email = 'nonrepeateble@mail.com' WHERE id = 1; COMMIT;`

TRX1:
`SELECT email FROM users WHERE id = 1; -- we will see nonrepeateble@mail.com`

4. `Phantom Reads`:

TRX1:
`BEGIN; SELECT email FROM users WHERE birth_date > '2000-01-01';`

TRX2:
`BEGIN; INSERT INTO users VALUES (null, 'new_user', 'phantom-reads@gmail.com', '2001-01-01'); COMMIT;`

TRX1:
`SELECT email FROM users WHERE birth_date > '2000-01-01'; -- we will see new row in range of results`

