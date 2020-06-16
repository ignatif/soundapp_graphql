# Migration `20200616065537-init`

This migration has been generated by ignatif at 6/16/2020, 6:55:37 AM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
PRAGMA foreign_keys=OFF;

CREATE TABLE "quaint"."User" (
"avatar" TEXT   ,"email" TEXT NOT NULL  ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"username" TEXT NOT NULL  )

CREATE TABLE "quaint"."Book" (
"authorUsername" TEXT   ,"description" TEXT NOT NULL  ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"image" TEXT   ,"name" TEXT NOT NULL  ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Chapter" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"content" TEXT NOT NULL  ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"image" TEXT   ,"title" TEXT   ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Voice" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"url" TEXT NOT NULL  ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Rating" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"stars" INTEGER NOT NULL  ,"voiceId" INTEGER   ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("voiceId") REFERENCES "Voice"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE UNIQUE INDEX "quaint"."User.username" ON "User"("username")

CREATE UNIQUE INDEX "quaint"."User.email" ON "User"("email")

PRAGMA "quaint".foreign_key_check;

PRAGMA foreign_keys=ON;
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration ..20200616065537-init
--- datamodel.dml
+++ datamodel.dml
@@ -1,0 +1,74 @@
+generator client {
+  provider = "prisma-client-js"
+}
+
+datasource db {
+  provider = "sqlite"
+  url      = "file:./dev.db"
+}
+
+
+model User {
+  id             Int        @default(autoincrement()) @id
+  username       String     @unique
+  email          String     @unique
+  avatar         String?
+  books          Book[]
+  chapters       Chapter[]
+  voices         Voice[]
+  ratings        Rating[]
+}
+
+model Book {
+  id             Int        @default(autoincrement()) @id
+  name           String
+  description    String
+  image          String?
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  chapters       Chapter[]
+  voices         Voice[]
+  ratings        Rating[]
+}
+
+model Chapter {
+  id             Int        @default(autoincrement()) @id
+  title          String?
+  content        String
+  image          String?
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  voices         Voice[]
+  ratings        Rating[]
+}
+
+model Voice {
+  id             Int        @default(autoincrement()) @id
+  url            String
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  ratings        Rating[]
+}
+
+model Rating {
+  id             Int        @default(autoincrement()) @id
+  stars          Int
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  voice          Voice?     @relation(fields: [voiceId], references: [id])
+  voiceId        Int?
+}
```

