Learn how to use the documentation feature in supastarter.

For the documentation feature of supastarter is also based on content-collections.

Create a documentation page
To create a new documentation page, simply create a new .mdx file in the content/documentation directory.


---
title: Documentation
subtitle: Learn how to use our software
---
 
Your content here
The frontmatter properties title and subtitle can be defined for each documentation page. The subtitle is optional.

Folder structure
The folder structure of the documentation is determined by the directory structure of the content/documentation directory:


content
└── documentation
    ├── getting-started
    │   └── index.mdx
    ├── installation
    │   ├── index.mdx
    |   └── linux.mdx
    ├── usage
    │   └── index.mdx
    ├── index.mdx
The index file of each directory will be used as the main page of the documentation section. The title of the directory will be used as the title of the documentation section.

If you want to customize the title or the order of directories / pages you can define a meta.json file in each directory:


{
  "items": {
    "getting-started": "Getting Started",
    "usage": "Usage",
    "installation": "Installation"
  }
}
The items object defines the order and the title of the directories / pages. If an item is not defined in the meta.json file, the title of the directory / page will be used and the order will be determined by the file system.

Multi-language support
The documentation feature also supports multi-language. To translate a documentation page, simply create a new .mdx file in the same directory and add the language code to the filename (like index.de.md for German):

index.de.mdx

---
title: Dokumentation
subtitle: Lerne, wie du unsere Software verwendest
---
 
Dein Inhalt hier
Make sure to use the same locales as defined in the i18n object in the config/index.ts file.

To translate the menu items of the documentation, you can define a meta.[locale].json file in each directory:

meta.de.json

{
  "items": {
    "getting-started": "Erste Schritte",
    "usage": "Verwendung",
    "installation": "Installation"
  }
}
