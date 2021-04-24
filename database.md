# Database

### SQLite

In the old days, there was just plain SQLite database. It wasn't exactly easy to use. One had to define multiple classes full of boilerplate code to fulfil basic CRUD operations. Furthermore, it was designed to be used with [`Cursors`](https://developer.android.com/reference/android/database/Cursor). `Cursors` aren't necessarily a bad concept, it just isn't natural to use it, since it doesn't return data as object, but one has to extract its values to compose an object first. Nowadays, it is discouraged to use SQLite directly. The recommended way is to use its wrapper - Room.

### Room

Room is a data persistence library which builds on top of SQLite and provides easy-to-use experience along with compile time schema verification.

