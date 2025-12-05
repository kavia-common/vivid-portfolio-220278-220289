# Database (MongoDB) - Portfolio App

This directory contains the MongoDB setup and helper scripts for the portfolio application. It is designed to run a local MongoDB instance and provide a consistent connection string for development.

Key points:
- Primary runtime port: 5001 (running container port as per orchestration)
- Local setup scripts default to port 5000 (see notes below)
- Always use db_connection.txt for connecting with mongosh
- Do NOT create .js/.json seed scripts; use mongosh one-liners instead

## Connection

Preferred method: read the provided connection command from db_connection.txt.

- File: db_connection.txt
- Example content:
  mongosh mongodb://appuser:dbuser123@localhost:5000/myapp?authSource=admin

Notes:
- The running container for this environment is exposed at port 5001. When connecting to the running service, substitute the port if needed:
  mongosh mongodb://appuser:dbuser123@localhost:5001/myapp?authSource=admin
- The startup.sh script writes db_connection.txt using port 5000 by default. If you are connecting to the already running container at 5001, use the 5001 port in your connection command.

Tip: If unsure, try the db_connection.txt command first; if it fails, try the same connection string but with port 5001.

## Database and Collections

Database name: myapp

Recommended collections:
- projects
- about
- skills
- messages
- media

### Suggested Schemas (informational)

These are reference structures to guide data shape; MongoDB is schemaless, so keep consistency in application code.

- projects:
  - _id: ObjectId
  - title: string
  - slug: string (unique)
  - description: string
  - tags: [string]
  - images: [string] (URLs or asset ids)
  - featured: boolean
  - createdAt: Date
  - updatedAt: Date
- about:
  - _id: ObjectId
  - name: string
  - title: string
  - bio: string
  - socials: { github: string, linkedin: string, twitter: string, email: string }
  - updatedAt: Date
- skills:
  - _id: ObjectId
  - category: string (e.g., "Frontend", "Backend", "DevOps")
  - items: [ { name: string, level: number } ]
  - updatedAt: Date
- messages:
  - _id: ObjectId
  - name: string
  - email: string
  - content: string
  - status: string (e.g., "new", "read", "archived")
  - createdAt: Date
- media:
  - _id: ObjectId
  - key: string (unique identifier)
  - url: string
  - type: string (e.g., "image", "video")
  - metadata: object
  - createdAt: Date

## Indexes

Create these indexes for performance and data integrity. Use mongosh one-liners.

- projects.slug unique
  db.projects.createIndex({ slug: 1 }, { unique: true })

- projects.featured
  db.projects.createIndex({ featured: 1 })

- messages.createdAt
  db.messages.createIndex({ createdAt: -1 })

- skills.category
  db.skills.createIndex({ category: 1 })

- media.key unique
  db.media.createIndex({ key: 1 }, { unique: true })

Run each command independently with mongosh -e or inside an interactive session.

## One-liner Seed Commands (mongosh)

CRITICAL: Do not create .js/.json files. Execute ONE command at a time.

First, connect (use db_connection.txt or adjust port 5001 as needed):
- Using db_connection.txt:
  $(cat db_connection.txt)
- Or explicitly on 5001:
  mongosh "mongodb://appuser:dbuser123@localhost:5001/myapp?authSource=admin"

Then run one-liners. Replace the connection string in -e examples as needed.

- Create collections (MongoDB creates on first write, but you can do explicit create):
  mongosh "mongodb://appuser:dbuser123@localhost:5001/myapp?authSource=admin" --eval 'db.createCollection("projects")'
  mongosh "..." --eval 'db.createCollection("about")'
  mongosh "..." --eval 'db.createCollection("skills")'
  mongosh "..." --eval 'db.createCollection("messages")'
  mongosh "..." --eval 'db.createCollection("media")'

- Insert sample about:
  mongosh "..." --eval 'db.about.insertOne({ name: "John Doe", title: "Creative Developer", bio: "I build animated, interactive experiences for the web.", socials: { github: "johndoe", linkedin: "john-doe", twitter: "johndoe", email: "john@example.com" }, updatedAt: new Date() })'

- Insert sample projects:
  mongosh "..." --eval 'db.projects.insertOne({ title: "Oceanic Portfolio", slug: "oceanic-portfolio", description: "Animated portfolio experience with modern visuals.", tags: ["react", "animation"], images: [], featured: true, createdAt: new Date(), updatedAt: new Date() })'
  mongosh "..." --eval 'db.projects.insertOne({ title: "Amber Studio", slug: "amber-studio", description: "Interactive studio microsite.", tags: ["node", "express"], images: [], featured: false, createdAt: new Date(), updatedAt: new Date() })'

- Insert sample skills:
  mongosh "..." --eval 'db.skills.insertOne({ category: "Frontend", items: [{ name: "React", level: 5 }, { name: "Framer Motion", level: 4 }], updatedAt: new Date() })'
  mongosh "..." --eval 'db.skills.insertOne({ category: "Backend", items: [{ name: "Node.js", level: 5 }, { name: "MongoDB", level: 4 }], updatedAt: new Date() })'

- Insert sample media:
  mongosh "..." --eval 'db.media.insertOne({ key: "hero-bg", url: "/media/hero-bg.jpg", type: "image", metadata: { w: 1920, h: 1080 }, createdAt: new Date() })'

- Create indexes (run individually):
  mongosh "..." --eval 'db.projects.createIndex({ slug: 1 }, { unique: true })'
  mongosh "..." --eval 'db.projects.createIndex({ featured: 1 })'
  mongosh "..." --eval 'db.messages.createIndex({ createdAt: -1 })'
  mongosh "..." --eval 'db.skills.createIndex({ category: 1 })'
  mongosh "..." --eval 'db.media.createIndex({ key: 1 }, { unique: true })'

Replace "..." with the full connection string you are using.

## Ports and Environment

- Running container (this environment): 5001
- Local bootstrap scripts (startup.sh, backup/restore): default to 5000
  - If you rely on startup.sh to provision users/db locally, it uses port 5000
  - When connecting to the deployed/running service for this work item, use 5001

Visualizer helper:
- db_visualizer/mongodb.env is generated by startup.sh and expects 5000 by default. Adjust port if using the running service (5001).

## Backup and Restore

Use the provided universal scripts. They auto-detect the running database.

- Backup:
  ./backup_db.sh
  - For MongoDB, produces database_backup.archive

- Restore:
  ./restore_db.sh
  - For MongoDB, restores from database_backup.archive with --drop

Notes:
- Ensure the target DB is reachable (mongosh ping) on the correct port (5000 locally via startup.sh or 5001 for running container) before running backup/restore.
- Scripts try multiple engines; MongoDB is detected via mongosh ping.

## Troubleshooting

- Connection refused:
  - Try port 5001 (running container) instead of 5000, or vice versa if using local startup.sh
  - Verify mongosh can ping: mongosh --port 5001 --eval "db.adminCommand('ping')"
- Auth issues:
  - Verify you are using authSource=admin in the URI
  - Confirm user appuser and password dbuser123 (or update to your environment)
- Visualizer:
  - source db_visualizer/mongodb.env
  - cd db_visualizer && npm start
  - Ensure MONGODB_URL points to the desired port (5000 local vs 5001 running container)

## Policy

- Do NOT add .js/.json seed files. Use mongosh one-liners.
- Execute database mutations one command at a time.
- Always prefer the connection string provided in db_connection.txt, adjusting only the port if needed for the running environment.
