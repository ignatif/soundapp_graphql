# Migration `20200702112422-books`

This migration has been generated by ignatif at 7/2/2020, 11:24:22 AM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
PRAGMA foreign_keys=OFF;

CREATE TABLE "quaint"."User" (
"avatar" TEXT   ,"email" TEXT   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"username" TEXT NOT NULL  )

CREATE TABLE "quaint"."Book" (
"authorUsername" TEXT   ,"description" TEXT NOT NULL  ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"image" TEXT   ,"name" TEXT NOT NULL  ,"views" INTEGER NOT NULL DEFAULT 0 ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Chapter" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"content" TEXT NOT NULL  ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"image" TEXT   ,"title" TEXT   ,"views" INTEGER NOT NULL DEFAULT 0 ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Tag" (
"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"label" TEXT NOT NULL  )

CREATE TABLE "quaint"."Comment" (
"authorUsername" TEXT   ,"body" TEXT NOT NULL  ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Review" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"message" TEXT   ,"stars" INTEGER NOT NULL  ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Like" (
"authorUsername" TEXT   ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"commentId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"reviewId" INTEGER   ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("commentId") REFERENCES "Comment"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("reviewId") REFERENCES "Review"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."Notification" (
"authorUsername" TEXT   ,"body" TEXT NOT NULL  ,"bookId" INTEGER   ,"chapterId" INTEGER   ,"id" INTEGER NOT NULL  PRIMARY KEY AUTOINCREMENT,"reviewId" INTEGER   ,FOREIGN KEY ("authorUsername") REFERENCES "User"("username") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("bookId") REFERENCES "Book"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("chapterId") REFERENCES "Chapter"("id") ON DELETE SET NULL ON UPDATE CASCADE,
FOREIGN KEY ("reviewId") REFERENCES "Comment"("id") ON DELETE SET NULL ON UPDATE CASCADE)

CREATE TABLE "quaint"."_UserFollows" (
"A" INTEGER NOT NULL  ,"B" INTEGER NOT NULL  ,FOREIGN KEY ("A") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE,
FOREIGN KEY ("B") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE)

CREATE TABLE "quaint"."_BookToTag" (
"A" INTEGER NOT NULL  ,"B" INTEGER NOT NULL  ,FOREIGN KEY ("A") REFERENCES "Book"("id") ON DELETE CASCADE ON UPDATE CASCADE,
FOREIGN KEY ("B") REFERENCES "Tag"("id") ON DELETE CASCADE ON UPDATE CASCADE)

CREATE TABLE "quaint"."_ChapterToTag" (
"A" INTEGER NOT NULL  ,"B" INTEGER NOT NULL  ,FOREIGN KEY ("A") REFERENCES "Chapter"("id") ON DELETE CASCADE ON UPDATE CASCADE,
FOREIGN KEY ("B") REFERENCES "Tag"("id") ON DELETE CASCADE ON UPDATE CASCADE)

CREATE UNIQUE INDEX "quaint"."User.username" ON "User"("username")

CREATE UNIQUE INDEX "quaint"."User.email" ON "User"("email")

CREATE UNIQUE INDEX "quaint"."_UserFollows_AB_unique" ON "_UserFollows"("A","B")

CREATE  INDEX "quaint"."_UserFollows_B_index" ON "_UserFollows"("B")

CREATE UNIQUE INDEX "quaint"."_BookToTag_AB_unique" ON "_BookToTag"("A","B")

CREATE  INDEX "quaint"."_BookToTag_B_index" ON "_BookToTag"("B")

CREATE UNIQUE INDEX "quaint"."_ChapterToTag_AB_unique" ON "_ChapterToTag"("A","B")

CREATE  INDEX "quaint"."_ChapterToTag_B_index" ON "_ChapterToTag"("B")

PRAGMA "quaint".foreign_key_check;

PRAGMA foreign_keys=ON;
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration ..20200702112422-books
--- datamodel.dml
+++ datamodel.dml
@@ -1,0 +1,120 @@
+generator client {
+  provider = "prisma-client-js"
+}
+
+datasource sqlite {
+  provider = "sqlite"
+  url = "***"
+}
+
+
+model User {
+  id             Int        @default(autoincrement()) @id
+  username       String     @unique
+  email          String?    @unique
+  avatar         String?
+  books          Book[]
+  chapters       Chapter[]
+  reviews        Review[]
+  likes          Like[]
+  comments       Comment[]
+  following      User[]     @relation("UserFollows", references: [id])
+  followedBy     User[]     @relation("UserFollows", references: [id])
+}
+
+model Book {
+  id             Int        @default(autoincrement()) @id
+  name           String
+  description    String
+  image          String?
+  views          Int        @default(0)
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  chapters       Chapter[]
+  tags           Tag[]      @relation(references: [id])
+  reviews        Review[]
+  likes          Like[]
+  comments       Comment[]
+}
+
+model Chapter {
+  id             Int        @default(autoincrement()) @id
+  title          String?
+  content        String
+  image          String?
+  views          Int        @default(0)
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  tags           Tag[]      @relation(references: [id])
+  reviews        Review[]
+  likes          Like[]
+  comments       Comment[]
+}
+
+model Tag {
+  id             Int        @default(autoincrement()) @id
+  label          String
+
+  books          Book[]     @relation(references: [id])
+  chapters       Chapter[]  @relation(references: [id])
+}
+
+model Comment {
+  id             Int        @default(autoincrement()) @id
+  body           String
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  likes          Like[]
+}
+
+model Review {
+  id             Int        @default(autoincrement()) @id
+  stars          Int
+  message        String?
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  likes          Like[]
+}
+
+model Like {
+  id             Int        @default(autoincrement()) @id
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  comment        Comment?   @relation(fields: [commentId], references: [id])
+  commentId      Int?
+  review         Review?    @relation(fields: [reviewId], references: [id])
+  reviewId       Int?
+}
+
+
+model Notification {
+  id             Int        @default(autoincrement()) @id
+  body           String
+
+  author         User?      @relation(fields: [authorUsername], references: [username])
+  authorUsername String?
+  book           Book?      @relation(fields: [bookId], references: [id])
+  bookId         Int?
+  chapter        Chapter?   @relation(fields: [chapterId], references: [id])
+  chapterId      Int?
+  review         Comment?   @relation(fields: [reviewId], references: [id])
+  reviewId       Int?
+}
```

