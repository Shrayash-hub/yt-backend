# User & Video Models, Password Hashing (bcrypt), and JWT Tokens

> This lesson writes the project's first two Mongoose schemas — **User** and **Video** — and introduces three major production concepts woven directly into those schemas: secure password hashing with **bcrypt**, authentication tokens with **JSON Web Tokens (JWT)**, and the **mongoose-aggregate-paginate-v2** plugin that unlocks MongoDB's advanced aggregation pipeline framework.

---

## 1. Overview

This lesson moves from pure setup into the project's **first real data models**. The `User` and `Video` schemas are deliberately built together because they're **tightly coupled** — every video needs an owner (a user), and every user's watch history is a list of video references.

**What gets built:**
1. `src/models/user.model.js` — the User schema, including username, email, avatar, cover image, watch history, password, and refresh token fields
2. `src/models/video.model.js` — the Video schema, including video file, thumbnail, title, description, duration, views, publish status, and owner
3. The **mongoose-aggregate-paginate-v2** plugin injected into the Video schema, enabling advanced aggregation pipeline queries later in the project
4. **Password security**: a Mongoose `pre("save")` hook that automatically hashes passwords using **bcrypt** before they're saved, plus a custom `isPasswordCorrect` method to verify a plaintext password against the stored hash
5. **JWT tokens**: custom `generateAccessToken` and `generateRefreshToken` methods added directly to the User schema, along with the environment variables needed to support them

**Practical purpose:** By the end of this lesson, the two foundational data models for the entire project exist, and the User model has built-in, reusable mechanisms for securely handling passwords and issuing authentication tokens — patterns every future authentication-related controller will rely on.

---

## 2. Key Concepts

### 2.1 Why User and Video Are Built Together
Although the full schema (visible in the project's shared Eraser/diagram link) includes many more models — tweets, subscriptions, comments, playlists, likes — this lesson focuses specifically on **User** and **Video** because they are **tightly coupled**: a video always has an owner (a user), and a user's watch history is fundamentally a list of video references. Building them together is called out as a practical team workflow point too — if assigning this work to a developer (or working on it yourself), it makes sense to handle both in the same branch/session.

### 2.2 MongoDB's Automatic `_id`
Unlike some other schema fields, an explicit `id` field is **not** written into the schema — MongoDB automatically generates a unique `_id` for every saved document, stored internally in BSON format (not plain JSON). This is flagged as something worth independently researching for a deeper understanding.

### 2.3 Storing Media as URLs, Not Files
Avatar images, cover images, video files, and thumbnails are **never stored directly in the database**. Instead:
- A **third-party service** (Cloudinary is named as the one this project will use, similar in purpose to AWS) handles the actual file upload.
- That service returns a **URL** pointing to the uploaded file.
- The database only stores that URL string.

**Why:** Storing large media files directly in a database creates heavy load and is explicitly called out as bad practice — even though MongoDB technically *allows* storing small files as binary media, professional codebases keep media storage separate.

### 2.4 The `owner` Field (Why Video Needs a User Reference)
Every video needs an `owner` field because, practically, whenever a video is displayed anywhere in the UI (a card, a watch history entry, etc.), you typically want to show **at least two pieces of information together**: something about the video itself (thumbnail, title) and something about who uploaded it. This is the direct reason the `owner` field exists on the Video schema, referencing back to the User model.

### 2.5 Watch History Design
The User schema includes a `watchHistory` field, designed as an **array of video IDs**. As a user watches videos, their IDs get **pushed** into this array — this is the mechanism by which the app will be able to reconstruct a user's full watch history later.

### 2.6 Schema Field Options Recap: `required`, `unique`, `lowercase`, `trim`, `index`
Several Mongoose schema field options are used throughout both models:
- **`required: true`** — the field must be present to save the document; can optionally include a **custom error message** (e.g., `[true, "Password is required"]`).
- **`unique: true`** — used for fields like `username` and `email`, which must not be duplicated across documents.
- **`lowercase: true`** — normalizes string input (e.g., usernames) to lowercase automatically.
- **`trim: true`** — removes leading/trailing whitespace automatically.
- **`index: true`** — marks a field as optimized for searching. Applied selectively (on `username` and `fullName` in this project) — **not** on every field, since indexing has a performance cost and should be used deliberately, only on fields you know will be searched frequently. This is explicitly called out as its own deep topic worth a dedicated future video.

### 2.7 mongoose-aggregate-paginate-v2 — Unlocking Aggregation Pipelines
A special plugin, **`mongoose-aggregate-paginate-v2`**, is installed and injected into the **Video** schema specifically (not the User schema).

**Why this matters:** Most beginner-level MongoDB/Mongoose tutorials only cover basic operations — `insertOne`, `insertMany`, `updateMany`, `findOne`, etc. The *real* power MongoDB offers in production — and what this project will lean on heavily — comes from its **aggregation framework**, which allows writing far more advanced, multi-stage database queries. This plugin specifically adds **pagination support on top of aggregation queries**, which isn't available out of the box.

> **Framed explicitly as a course differentiator:** the instructor states this level of aggregation pipeline usage isn't something typically taught in paid or free courses, and is meant to push the project to an "advanced" level.

### 2.8 Mongoose Plugins (Conceptual)
Mongoose allows two relevant extension mechanisms used in this lesson:
- **Hooks/middleware** (e.g., `pre`, `post`) — run custom code immediately before or after certain operations (like saving a document).
- **Plugins** — reusable pieces of functionality (like `mongoose-aggregate-paginate-v2`) that can be injected into a schema to extend its capabilities, using `schema.plugin(...)`.

### 2.9 Why Passwords Are Never Stored in Plain Text
Storing a password as plain text (e.g., literally storing `"1234"`) is explicitly called out as something that should never be done — because databases do sometimes leak, and a plaintext password leak is far more damaging than an encrypted/hashed one.

**The challenge this creates:** if a password is encrypted/hashed before storage, you can no longer directly compare an incoming login attempt to the stored value as plain strings — you need a way to verify a plaintext password against its hashed counterpart **without ever decrypting it**. This is exactly what `bcrypt`'s hashing and comparison functions solve.

### 2.10 bcrypt vs bcryptjs
Two related libraries are discussed:
- **`bcrypt`** — built as a package on top of Node.js's native C++ bindings.
- **`bcryptjs`** — described as optimized JavaScript with zero dependencies, compatible with `bcrypt`.

Both are widely used and functionally near-identical (same hash generation, same configuration approach). This project uses **core `bcrypt`**. The choice between them is explicitly framed as a matter of preference/team convention rather than a technical necessity.

**What bcrypt does:** *"It helps you hash your password"* — solving the plain-text storage problem described above.

### 2.11 JWT (JSON Web Tokens) — Conceptual Foundation
**JWT is described as a *bearer token*** — a concept the instructor specifically flags as a common interview question.

**"Bearer" meaning:** whoever *bears* (holds/possesses) this token is trusted — i.e., the system will send data to whoever presents a valid token, much like a physical key. This is why token security matters: losing a token is like losing a key, even though the underlying cryptography is strong.

**Structure of a JWT** (referencing jwt.io as a visualization tool):
1. **Header** — automatically injected metadata, including which algorithm is used.
2. **Payload** — the actual data being encoded (e.g., user ID, email) — described as a "fancy name" for the data section; whatever you choose to include gets encoded here.
3. **Signature** — generated using a cryptographic algorithm and a **secret**.

**The role of the secret:** the secret is what makes each token unique and secure — the signing algorithm itself is publicly known, so without a secret, anyone could decode and forge tokens. The secret is what protects the payload's integrity.

### 2.12 Access Token vs Refresh Token
Two JWTs are used in this project, both generated using the same underlying mechanism but serving different purposes:

| | Access Token | Refresh Token |
|---|---|---|
| **Payload contents** | More information: `_id`, `email`, `username`, `fullName` | Minimal information: just `_id` |
| **Expiry** | Shorter (e.g., 1 day in this project) | Longer (e.g., 10 days in this project) |
| **Stored in database?** | **No** — not persisted in the User document | **Yes** — stored directly in the User schema's `refreshToken` field |

> **Why only the refresh token is stored in the schema:** the instructor notes this project will use both sessions and cookies with strong security practices, and explicitly defers the full working mechanics of access vs. refresh tokens (why one is persisted and the other isn't, how refresh flows work) to a dedicated future lesson with diagrams — this lesson only covers *how to generate* both tokens.

### 2.13 Environment Variables for Tokens
Four new environment variables are introduced, following a consistent `ALL_CAPS_WITH_UNDERSCORES` naming convention (explicitly noted as a **standard practice convention**, not a hard technical requirement):
```env
ACCESS_TOKEN_SECRET=<long random string>
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=<a different long random string>
REFRESH_TOKEN_EXPIRY=10d
```

### 2.14 Mongoose `pre` Hooks (Middleware) — Used for Password Hashing
A **`pre("save")` hook** is used to automatically run code **immediately before** a document is saved to the database.

**Critical detail — regular function, not arrow function:** the callback passed to `pre` must be a **regular `function`**, not an arrow function, because arrow functions don't bind their own `this` context. Inside the hook, `this` needs to refer to the specific document being saved (to access and modify its fields, like `this.password`) — an arrow function would not provide that context correctly.

**The "always-rehashing" problem and its fix:** without an additional check, this hook would re-hash the password **every single time** the document is saved for *any* reason (e.g., just updating an avatar) — which is incorrect. The fix is to check **whether the password field specifically was modified** before running the hashing logic, using Mongoose's built-in `this.isModified("password")` check. If the password field wasn't touched in this particular save operation, the hook exits early via `return next()` without re-hashing.

### 2.15 Mongoose Custom Methods
Beyond hooks, Mongoose also allows injecting **custom methods** directly onto schema instances via the schema's `.methods` object — the same general mechanism conceptually as adding hooks, but for instance methods you can call directly on a retrieved user document (e.g., `user.isPasswordCorrect(...)`).

Three custom methods are added to the User schema in this lesson:
1. `isPasswordCorrect(password)` — compares a plaintext password against the stored hash using `bcrypt.compare(...)`
2. `generateAccessToken()` — issues a short-lived JWT containing more user detail
3. `generateRefreshToken()` — issues a longer-lived JWT containing minimal user detail (just the ID)

---

## 3. Code Breakdown

### Step 1 — File Setup
```bash
# inside src/models/
touch user.model.js
touch video.model.js
```

### Step 2 — Installing Required Packages
```bash
npm install mongoose-aggregate-paginate-v2 bcrypt jsonwebtoken
```

### Step 3 — Basic User Schema Skeleton
**File: `src/models/user.model.js`**
```javascript
import mongoose, { Schema } from "mongoose";

const userSchema = new Schema(
  {
    // fields go here
  },
  { timestamps: true }
);

export const User = mongoose.model("User", userSchema);
```
**What's happening, line by line:**
- `mongoose` is imported, and `Schema` is **destructured** directly from it — a shortcut so `mongoose.Schema` doesn't need to be written repeatedly.
- `new Schema({...}, {...})` creates the schema definition — the first argument is the object of fields, the second is an options object.
- `{ timestamps: true }` automatically adds and manages `createdAt` and `updatedAt` fields — no manual definition needed.
- `mongoose.model("User", userSchema)` asks Mongoose to build an actual usable **model** based on this schema, naming it `"User"` (capitalized, singular — a standard Mongoose convention; Mongoose automatically pluralizes/lowercases this internally when naming the actual MongoDB collection).
- `export const User = ...` exports the model so it can be imported and used (e.g., `User.create(...)`, `User.findOne(...)`) elsewhere in the app.

### Step 4 — Filling In the User Schema Fields
```javascript
const userSchema = new Schema(
  {
    username: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
      index: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
      trim: true,
    },
    fullName: {
      type: String,
      required: true,
      trim: true,
      index: true,
    },
    avatar: {
      type: String, // cloudinary URL
      required: true,
    },
    coverImage: {
      type: String, // cloudinary URL
    },
    watchHistory: [
      {
        type: Schema.Types.ObjectId,
        ref: "Video",
      },
    ],
    password: {
      type: String,
      required: [true, "Password is required"],
    },
    refreshToken: {
      type: String,
    },
  },
  { timestamps: true }
);
```
**What's happening, field by field:**
- **`username`** — a required, unique, lowercase, trimmed string, additionally marked with `index: true` since it'll be frequently searched.
- **`email`** — required, unique, lowercase, trimmed — but **not indexed**, since indexing every field is explicitly called wasteful; only fields expected to be searched often should carry that cost.
- **`fullName`** — required and trimmed (not lowercased, since names shouldn't be forced to lowercase), and **is** indexed here, since the project's data design anticipates searching users by name.
- **`avatar`** — a required string storing a Cloudinary URL (not the image itself).
- **`coverImage`** — a string storing a Cloudinary URL, but **not required** — explicitly left as something the frontend can handle the absence of gracefully.
- **`watchHistory`** — an **array** of objects, each shaped as `{ type: Schema.Types.ObjectId, ref: "Video" }`. This tells Mongoose: each entry in this array is a reference (by ObjectId) to a document in the `Video` collection.
- **`password`** — a required string, using the **array syntax** `[true, "Password is required"]` to attach a custom error message alongside the `required` validation.
- **`refreshToken`** — a plain string field, used later to persist the user's current refresh token.

### Step 5 — Basic Video Schema Skeleton with the Aggregate-Paginate Plugin
**File: `src/models/video.model.js`**
```javascript
import mongoose, { Schema } from "mongoose";
import mongooseAggregatePaginate from "mongoose-aggregate-paginate-v2";

const videoSchema = new Schema(
  {
    // fields go here
  },
  { timestamps: true }
);

videoSchema.plugin(mongooseAggregatePaginate);

export const Video = mongoose.model("Video", videoSchema);
```
**What's happening, line by line:**
- `mongoose` and `Schema` are imported the same way as in the User model.
- `mongooseAggregatePaginate` is imported from the newly installed plugin package.
- After the schema is defined, `videoSchema.plugin(mongooseAggregatePaginate)` **injects** the plugin's capabilities into this specific schema — enabling aggregation-pipeline-based queries with pagination support on the `Video` model going forward.
- The model is then created and exported the same way as `User`.

### Step 6 — Filling In the Video Schema Fields
```javascript
const videoSchema = new Schema(
  {
    videoFile: {
      type: String, // cloudinary URL
      required: true,
    },
    thumbnail: {
      type: String, // cloudinary URL
      required: true,
    },
    title: {
      type: String,
      required: true,
    },
    description: {
      type: String,
      required: true,
    },
    duration: {
      type: Number, // from cloudinary
      required: true,
    },
    views: {
      type: Number,
      default: 0,
    },
    isPublished: {
      type: Boolean,
      default: true,
    },
    owner: {
      type: Schema.Types.ObjectId,
      ref: "User",
    },
  },
  { timestamps: true }
);
```
**What's happening, field by field:**
- **`videoFile`** — required string, Cloudinary URL for the actual video file.
- **`thumbnail`** — required string, Cloudinary URL for the video's thumbnail image.
- **`title`** and **`description`** — required strings, both user-provided (not derived from Cloudinary).
- **`duration`** — a `Number`, explicitly noted as **not user-provided** — Cloudinary returns this automatically as metadata when a file finishes uploading.
- **`views`** — a `Number` defaulting to `0`. A default value is necessary here since views shouldn't arbitrarily start as anything else, and the field will be incremented over time as the video is watched.
- **`isPublished`** — a `Boolean` flag defaulting to `true`, controlling whether the video is currently visible/available to the public.
- **`owner`** — references the `User` model via `Schema.Types.ObjectId` and `ref: "User"`, identifying who uploaded the video.

### Step 7 — Password Hashing with a `pre("save")` Hook
Added inside `user.model.js`, after the schema definition (before exporting the model):
```javascript
import bcrypt from "bcrypt";

userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();

  this.password = await bcrypt.hash(this.password, 10);
  next();
});
```
**What's happening, line by line:**
- `bcrypt` is imported.
- `userSchema.pre("save", async function (next) {...})` registers a hook that runs **immediately before** any `save()` call on a User document. Note the **regular `function` keyword**, not an arrow function — required so `this` correctly refers to the document instance being saved.
- `if (!this.isModified("password")) return next();` — this is the guard against re-hashing an already-hashed password on every save. `this.isModified("password")` returns `true` only if the `password` field was specifically changed in this particular save operation. If it wasn't modified, the hook immediately calls `next()` and exits, skipping the hashing logic entirely.
- `this.password = await bcrypt.hash(this.password, 10);` — if the password *was* modified, it's hashed using `bcrypt.hash()`, passing the plaintext password and a **cost factor of `10`** (the number of hashing rounds/salt rounds), and the result overwrites the plaintext value on `this.password`.
- `next();` — called at the end, signaling that the hook's work is done and the save operation can proceed.

### Step 8 — `isPasswordCorrect` Custom Method
Added via `userSchema.methods`:
```javascript
userSchema.methods.isPasswordCorrect = async function (password) {
  return await bcrypt.compare(password, this.password);
};
```
**What's happening, line by line:**
- `userSchema.methods.isPasswordCorrect = async function (password) {...}` — adds a new instance method called `isPasswordCorrect` to every document created from this schema. Again, a **regular function** is used (not an arrow function) so `this` refers to the specific user document the method is called on.
- Inside, `bcrypt.compare(password, this.password)` is called: the first argument is the **plaintext** password (e.g., what a user typed in during login), and the second is the **hashed** password already stored on this document (`this.password`).
- `bcrypt.compare(...)` internally re-applies the same hashing logic to the plaintext input and checks whether it matches the stored hash — without ever reversing/decrypting the stored hash.
- The method is `async` and the result is `await`-ed, since the comparison involves cryptographic computation.
- The function returns the boolean result of `bcrypt.compare(...)` directly — `true` if the passwords match, `false` otherwise.

**Usage pattern (for future controllers):**
```javascript
const isCorrect = await user.isPasswordCorrect(enteredPassword);
```

### Step 9 — Token Environment Variables
In `.env` (and `.env.sample`):
```env
ACCESS_TOKEN_SECRET=shrey-aur-code
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=shrey-aur-backend
REFRESH_TOKEN_EXPIRY=10d
```
*(Real projects should use long, random, cryptographically strong strings here — not simple words; the simple placeholders above mirror what's typed in the video for clarity, with an explicit note that production secrets should be properly randomized.)*

### Step 10 — `generateAccessToken` and `generateRefreshToken` Custom Methods
```javascript
import jwt from "jsonwebtoken";

userSchema.methods.generateAccessToken = function () {
  return jwt.sign(
    {
      _id: this._id,
      email: this.email,
      username: this.username,
      fullName: this.fullName,
    },
    process.env.ACCESS_TOKEN_SECRET,
    {
      expiresIn: process.env.ACCESS_TOKEN_EXPIRY,
    }
  );
};

userSchema.methods.generateRefreshToken = function () {
  return jwt.sign(
    {
      _id: this._id,
    },
    process.env.REFRESH_TOKEN_SECRET,
    {
      expiresIn: process.env.REFRESH_TOKEN_EXPIRY,
    }
  );
};
```
**What's happening, line by line:**
- `jwt` (from the `jsonwebtoken` package) is imported.
- **`generateAccessToken`** — a regular (non-async) function, since signing a token here doesn't involve meaningfully slow computation in this project's experience (the instructor notes `async` wasn't found necessary in testing, though acknowledges this could vary).
  - `jwt.sign(payload, secret, options)` is called with three arguments:
    - **Payload object** — `_id`, `email`, `username`, and `fullName`, all pulled directly off `this` (the current user document).
    - **Secret** — `process.env.ACCESS_TOKEN_SECRET`.
    - **Options object** — `{ expiresIn: process.env.ACCESS_TOKEN_EXPIRY }`, setting the token's expiry.
  - The signed JWT string is returned directly.
- **`generateRefreshToken`** — structurally identical, but with a **much smaller payload** (only `_id`) and its own separate secret/expiry environment variables (`REFRESH_TOKEN_SECRET`, `REFRESH_TOKEN_EXPIRY`), reflecting its different purpose (longer-lived, less sensitive data exposure).

### Step 11 — Commit and Push
```bash
git add .
git commit -m "add user and video model"
git push
```

### Flow Summary
```
Create user.model.js and video.model.js
       ↓
Install mongoose-aggregate-paginate-v2, bcrypt, jsonwebtoken
       ↓
Define userSchema fields (username, email, fullName, avatar, coverImage, watchHistory, password, refreshToken)
       ↓
Define videoSchema fields (videoFile, thumbnail, title, description, duration, views, isPublished, owner)
       ↓
Inject mongoose-aggregate-paginate-v2 into videoSchema via .plugin()
       ↓
Add userSchema.pre("save") hook → hash password ONLY if modified, using bcrypt.hash()
       ↓
Add userSchema.methods.isPasswordCorrect → bcrypt.compare() plaintext vs. stored hash
       ↓
Add ACCESS_TOKEN_* and REFRESH_TOKEN_* env vars
       ↓
Add userSchema.methods.generateAccessToken (larger payload, shorter expiry)
       ↓
Add userSchema.methods.generateRefreshToken (minimal payload, longer expiry)
       ↓
Commit and push
```

---

## 4. Functions & Methods

### `new Schema(definition, options)`
**Problem it solves:** Defines the shape, types, and validation rules for a MongoDB collection's documents, which Mongoose then uses to build an actual usable model.

**Syntax:**
```javascript
const mySchema = new Schema(
  { fieldName: { type: String, required: true } },
  { timestamps: true }
);
```

**Parameters:**
- `definition` *(object, required)* — maps field names to their type and validation configuration (e.g., `type`, `required`, `unique`, `default`, `ref`, etc.).
- `options` *(object, optional)* — schema-level settings; this project uses `{ timestamps: true }` to auto-manage `createdAt`/`updatedAt`.

**Return value:** A `Schema` instance, ready to be passed into `mongoose.model(...)`.

---

### `mongoose.model(name, schema)`
**Problem it solves:** Converts a schema definition into an actual usable Mongoose **model** — the object you call methods like `.create()`, `.find()`, `.findById()` on.

**Syntax:**
```javascript
const ModelName = mongoose.model(<modelName>, <schema>);
```

**Parameters:**
- `<modelName>` *(string)* — the model's name, conventionally **capitalized and singular** (e.g., `"User"`, `"Video"`). Mongoose automatically derives the actual MongoDB collection name from this (typically lowercased and pluralized).
- `<schema>` *(Schema instance)* — the schema this model is based on.

**How it works internally:** Mongoose compiles the schema into a constructor function with built-in methods (`save`, `findOne`, etc.) plus any custom hooks/methods/plugins attached to that schema.

**Return value:** A Mongoose `Model`, exported and used throughout the app to interact with that collection.

---

### `schema.plugin(pluginFunction)`
**Problem it solves:** Extends a schema's capabilities by injecting reusable, pre-built functionality — in this case, adding aggregation-pipeline pagination support that Mongoose doesn't include by default.

**Syntax:**
```javascript
videoSchema.plugin(mongooseAggregatePaginate);
```

**Parameters:**
- `pluginFunction` *(function)* — the plugin module being applied, here imported from `mongoose-aggregate-paginate-v2`.

**How it works internally:** The plugin function receives the schema and attaches additional static methods (in this case, an `aggregatePaginate` method) directly onto the resulting model, without modifying the schema's field definitions.

**Return value:** No meaningful return value used here — this is a side-effecting call that modifies the schema in place.

---

### `schema.pre(hookName, callback)`
**Problem it solves:** Runs custom logic automatically **before** a specified Mongoose operation occurs — used here to hash a password right before a document is saved, without requiring every controller to remember to do this manually.

**Syntax:**
```javascript
userSchema.pre("save", function (next) {
  // logic here
  next();
});
```

**Parameters:**
- `hookName` *(string)* — which operation to hook into, e.g., `"save"`. Other supported hooks include `"validate"`, `"remove"`, `"updateOne"`, `"deleteOne"`, etc.
- `callback` *(function)* — **must be a regular `function`, not an arrow function**, so that `this` correctly refers to the document instance being operated on. Receives a `next` callback as its parameter (and can be declared `async` to use `await` inside).

**How it works internally:** Mongoose executes all registered `pre` hooks for an operation, in the order they were defined, **before** actually performing that operation. Each hook must call `next()` (or, if async/returns a Promise, simply resolve) to signal completion and allow the next hook (or the actual operation) to proceed.

**Return value:** No direct return value used by the caller; the hook's effect is via mutating `this` (the document) and calling `next()`.

---

### `document.isModified(fieldName)`
**Problem it solves:** Checks whether a specific field was changed in the **current** save operation, preventing unnecessary or incorrect logic (like re-hashing an already-hashed password) from running on every save.

**Syntax:**
```javascript
this.isModified(<fieldName>);
```

**Parameters:**
- `<fieldName>` *(string)* — the name of the field to check, e.g., `"password"`. Must be passed as a string.

**How it works internally:** Mongoose internally tracks which fields have been changed on a document instance since it was last saved or fetched; `isModified()` simply checks this internal tracking for the given field name.

**Return value:** A `boolean` — `true` if the specified field was modified in this operation, `false` otherwise.

---

### `bcrypt.hash(data, saltRounds)`
**Problem it solves:** Converts a plaintext password into a secure, one-way hashed string suitable for database storage — so that even if the database is compromised, the original password isn't directly exposed.

**Syntax:**
```javascript
await bcrypt.hash(<plaintextString>, <saltRounds>);
```

**Parameters:**
- `<plaintextString>` *(string, required)* — the raw value to hash, e.g., `this.password` before hashing.
- `<saltRounds>` *(number, required)* — how many hashing rounds (cost factor) to apply, e.g., `10`. Higher numbers increase security but also increase computation time.

**How it works internally:** Returns a Promise (hence `await`-ed) that resolves once the cryptographic hashing algorithm completes — described in the lesson as computationally non-trivial, which is why these functions are typically written as `async`.

**Return value:** A `Promise<string>` resolving to the hashed password string, ready to be assigned and saved.

---

### `bcrypt.compare(plaintext, hashedValue)`
**Problem it solves:** Verifies whether a plaintext input (e.g., a login attempt) matches a previously hashed value, **without ever reversing/decrypting** the stored hash.

**Syntax:**
```javascript
await bcrypt.compare(<plaintextString>, <hashedString>);
```

**Parameters:**
- `<plaintextString>` *(string, required)* — the raw value to check, e.g., a password entered during login.
- `<hashedString>` *(string, required)* — the previously stored hash to compare against, e.g., `this.password` as retrieved from the database.

**How it works internally:** Internally re-applies the same hashing process to the plaintext input (using the salt embedded in the stored hash) and compares the result to the stored hash — this is why the original plaintext is never needed, only re-derivable via the same algorithm.

**Return value:** A `Promise<boolean>` — resolves to `true` if the values match, `false` otherwise.

---

### `schema.methods.<methodName> = function (...) {...}`
**Problem it solves:** Adds a **custom, reusable instance method** directly onto documents created from a schema — allowing logic like password verification or token generation to be called naturally on a user object (e.g., `user.isPasswordCorrect(...)`), rather than as a separate standalone utility function.

**Syntax:**
```javascript
schema.methods.methodName = function (param1, param2) {
  // can access `this` (the current document)
};
```

**Parameters:** Whatever parameters the specific method needs (e.g., `isPasswordCorrect(password)` takes one parameter; `generateAccessToken()` takes none, relying entirely on `this`).

**How it works internally:** Mongoose attaches whatever is assigned to `schema.methods.<name>` onto every document instance created from that schema, as a callable method with `this` correctly bound to that specific document — **provided a regular `function` is used**, not an arrow function.

**Return value:** Whatever the specific method explicitly returns — varies per method (a boolean for `isPasswordCorrect`, a signed JWT string for the token-generating methods).

---

### `jwt.sign(payload, secretOrPrivateKey, options)`
**Problem it solves:** Generates a signed JSON Web Token — a secure, verifiable, encoded string that can later be checked to confirm both its authenticity (it wasn't tampered with) and to extract the data embedded in its payload.

**Syntax:**
```javascript
jwt.sign(<payloadObject>, <secret>, <optionsObject>);
```

**Parameters:**
- `<payloadObject>` *(object, required)* — the data to embed in the token, e.g., `{ _id: this._id, email: this.email, ... }`. Whatever is included here becomes readable (though not editable without invalidating the signature) by anyone who decodes the token.
- `<secret>` *(string, required)* — the cryptographic secret used to sign the token, e.g., `process.env.ACCESS_TOKEN_SECRET`. This is what makes the token unguessable/unforgeable without knowledge of the secret.
- `<optionsObject>` *(object, optional)* — additional signing configuration; this project uses `{ expiresIn: process.env.ACCESS_TOKEN_EXPIRY }` to control how long the token remains valid (e.g., `"1d"`, `"10d"`).

**How it works internally:** Combines the header (algorithm metadata, auto-generated), the encoded payload, and a cryptographic signature (derived from the payload + secret using the specified algorithm) into the familiar three-part JWT string format, viewable/decodable (though not forgeable without the secret) via tools like jwt.io.

**Return value:** A signed JWT `string`.

---

## 5. Logic Explanation

The reasoning behind this lesson's structure builds carefully, concept by concept:

1. **Build coupled models together, deliberately.** User and Video are introduced in the same lesson specifically because they reference each other (`owner` on Video, `watchHistory` on User) — building them in isolation would mean writing references to a model that doesn't exist yet, or coming back later to retrofit connections. This also mirrors a realistic team workflow consideration (assigning related, coupled work as a single unit).

2. **Separate concerns between schema *fields* and schema *behavior*.** The lesson clearly distinguishes between defining what data a document holds (fields like `username`, `password`) and defining what a document can *do* (hooks like password hashing, methods like token generation) — these are taught as two distinct categories of Mongoose schema capability, each solving a different kind of problem.

3. **Treat password storage as a problem requiring its own dedicated solution, not an afterthought.** Rather than simply marking the `password` field `required` and moving on, the lesson explicitly frames plaintext password storage as a real security risk, then walks through *why* that creates a follow-up problem (you can't string-compare a hash to plaintext), motivating the introduction of `bcrypt` specifically to solve that two-sided problem (hash on save, compare on login).

4. **Use Mongoose's `pre("save")` hook so password hashing is automatic and impossible to forget.** Rather than requiring every future controller that creates or updates a user to remember to manually hash the password, the hook approach guarantees this happens centrally, every time, regardless of which part of the codebase triggers the save.

5. **Guard the hook with `isModified("password")` to avoid an unintended side effect.** This is presented as solving a problem the naive version of the hook would silently introduce: without the check, *any* save operation (even an unrelated one, like updating an avatar) would re-hash an already-hashed password, corrupting it. The fix — checking specifically whether the password field itself was the one modified — is shown as the necessary, deliberate correction.

6. **Insist on regular functions over arrow functions specifically where `this` context matters.** This is called out explicitly, twice (once for the `pre` hook, once for custom methods) — because Mongoose's hook/method system depends on `this` being dynamically bound to the specific document instance, which arrow functions structurally cannot provide. This is taught as a concrete, practical consequence of a JavaScript fundamental (lexical `this` binding in arrow functions) rather than an arbitrary rule.

7. **Introduce JWTs by first explaining the concept ("bearer token"), before writing any code.** Mirroring the approach used for middleware in the previous lesson, the abstract idea (whoever holds the token is trusted; the secret is what makes it secure) is explained before the `jwt.sign()` syntax — reinforcing the series' general pattern of concept-first, code-second.

8. **Distinguish access and refresh tokens by *purpose*, reflected directly in their payload size and expiry.** The access token carries more user detail and expires quickly, since it's used frequently for short-term authorization. The refresh token carries minimal detail (just an ID) and lives longer, since its only job is to be exchanged for a new access token later — and only the refresh token is persisted in the database, since the access token's short lifespan and lack of persistence is itself part of the security model (though the *mechanics* of that exchange are explicitly deferred to a future lesson).

9. **Defer full token *usage* mechanics on purpose, while still teaching full token *generation* mechanics now.** The instructor is explicit that this lesson only covers how to generate both tokens as reusable schema methods — the actual login/refresh flow, with diagrams, is intentionally left for a dedicated future lesson, consistent with the series' broader pattern of breaking complex topics into focused, digestible pieces rather than front-loading everything at once.

10. **Introduce the aggregation pipeline plugin early, even though it won't be used until later.** Injecting `mongoose-aggregate-paginate-v2` into the Video schema now — well before any aggregation queries are actually written — sets up the model to support that advanced functionality from the start, rather than requiring a schema modification later once aggregation queries become necessary.

---

## 6. Problem It Solves

**The core problem:** A backend application needs a reliable, secure way to (a) define and persist its core data structures, (b) protect sensitive user data like passwords, and (c) issue trustworthy proof of identity (tokens) for authenticated requests — all of which, if handled inconsistently or naively, create serious security and maintainability risks.

**How this lesson solves it:**
- **`User` and `Video` Mongoose schemas** → establish the project's two foundational, interrelated data structures, with field-level validation (`required`, `unique`, `trim`, `lowercase`) ensuring data consistency at the database layer itself, not just in application code.
- **Selective `index: true` usage** → optimizes specifically the fields expected to be searched often (`username`, `fullName`), rather than indiscriminately indexing everything, balancing search performance against the real cost indexing introduces.
- **Media stored as URLs, not files** → keeps the database lightweight and offloads actual file storage/serving to a purpose-built third-party service (Cloudinary), following standard production practice.
- **`mongoose-aggregate-paginate-v2` plugin** → unlocks MongoDB's advanced aggregation framework (well beyond basic CRUD operations) for the Video model, setting up the infrastructure needed for the complex, production-grade queries promised later in the series.
- **`bcrypt` hashing via a `pre("save")` hook, guarded by `isModified("password")`** → guarantees passwords are never stored in plaintext, that hashing happens automatically and consistently regardless of which controller triggers a save, and that this hashing logic doesn't incorrectly re-run on unrelated updates.
- **`isPasswordCorrect` via `bcrypt.compare`** → provides a safe, built-in way to verify login attempts without ever needing to reverse the stored hash.
- **`generateAccessToken` / `generateRefreshToken` via `jwt.sign`** → gives the User model a reusable, centralized mechanism for issuing properly structured, appropriately scoped (in payload size and expiry) authentication tokens — laying the groundwork for the login/session system to be built in upcoming lessons.

In short: this lesson solves the problem of building **secure-by-default, production-shaped data models** — where sensitive operations (password handling, token issuance) are baked directly into the schema itself via hooks and methods, rather than left to be remembered (and potentially forgotten or inconsistently implemented) by whoever writes a future controller.

---

## 7. Approach for Future Documentation

*(Carried forward and extended from prior lessons — update this section, don't duplicate it, as new patterns emerge.)*

### 7.1 Template Reminder
Continue using the standard structure: **Overview → Key Concepts → Code Breakdown → Functions & Methods → Logic Explanation → Problem It Solves → Approach for Future Documentation → What's Next.**

### 7.2 New Habits Introduced by This Lesson — Carry Forward
- **Document field-by-field schema breakdowns as tables or structured lists when the field count is high.** This lesson's User and Video schemas each have 7+ fields with varying options (`required`, `unique`, `index`, `default`, `ref`) — documenting each field's *specific* configuration (not just "it has validation") preserves the reasoning behind each individual choice, which the video itself explains field by field.
- **Flag JavaScript-fundamentals-driven implementation details explicitly**, especially where getting it wrong silently breaks things. The "regular function vs. arrow function for `this` binding" point recurs twice in this single lesson (hooks and methods) — future documentation should keep highlighting these moments as their own callouts, since they're easy to miss and cause confusing bugs if overlooked.
- **Document "naive version → discovered problem → fix" sequences as a named pattern**, not just the final correct code. This lesson's password-hashing hook is a clear example: the *first* version (hash on every save) is shown as flawed before the `isModified` guard is introduced as the fix. This mirrors the same pedagogical pattern seen in earlier lessons (e.g., the two database-connection approaches) and should continue to be preserved as its own mini-narrative, not collapsed into just the final working code.
- **Explicitly track what's deferred vs. what's complete**, especially for multi-part systems like authentication. This lesson generates tokens but does not yet implement their full usage flow (login, refresh-token exchange, cookie/session handling) — future documentation must clearly mark this boundary so readers don't assume authentication is "done" after this lesson.
- **Use comparison tables for closely related concepts with meaningful differences** (e.g., the Access Token vs. Refresh Token table in this README). This format made the payload/expiry/persistence differences immediately scannable, and should be reused whenever a lesson introduces two parallel-but-distinct mechanisms.

### 7.3 Running Glossary — Additions from This Lesson
*(Append to the cumulative glossary maintained across the series)*
- Mongoose `Schema`, `mongoose.model()`, `schema.plugin()`
- `mongoose-aggregate-paginate-v2`, MongoDB aggregation pipelines (conceptually introduced, not yet used)
- Schema field options: `required` (with custom messages), `unique`, `lowercase`, `trim`, `index`, `default`, `ref`, `Schema.Types.ObjectId`
- `schema.pre("save")`, `document.isModified(field)`
- `schema.methods.<name>`
- `bcrypt`, `bcrypt.hash()`, `bcrypt.compare()`, salt rounds
- JWT, bearer tokens, `jwt.sign()`, payload/header/signature structure
- Access token vs. refresh token (purpose, payload size, expiry, persistence differences)

### 7.4 Anticipated Structure for Future Lessons
Based on what this lesson explicitly defers, future documentation should anticipate (and cross-reference back to this lesson when it arrives):
- A dedicated, diagram-supported lesson on **how access and refresh tokens actually work together** in a login/session flow — including why only the refresh token is persisted, and how token refreshing is triggered.
- Discussion of **cookies and sessions** in more depth (mentioned here as both being used "with strong security" but not yet elaborated).
- The **actual use** of `mongoose-aggregate-paginate-v2` once real aggregation pipeline queries are written — this lesson only sets up the plugin, it doesn't yet use it.

---

## What's Next

Based on the lesson's own framing, upcoming videos will:
- Build out the actual **authentication flow** (login, registration, token issuance/verification in real controllers), using the schema methods defined here
- Provide a **dedicated, diagram-based explanation** of how access tokens and refresh tokens work together in practice
- Begin writing **aggregation pipeline queries** against the Video model, making use of the `mongoose-aggregate-paginate-v2` plugin set up in this lesson
- Continue the project's broader pattern of introducing one focused concept per lesson, building toward the full-featured video platform described in the series' introduction

> This README documents the **data modeling phase** of the project — specifically the User and Video schemas, along with the password-security and token-generation mechanisms built directly into the User model. No controllers, routes, or actual authentication endpoints have been written yet; this lesson only establishes the underlying data structures and reusable schema-level utilities those future pieces will depend on.