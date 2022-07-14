# Markdown indexed block

This lib will annotate every block in the markdown file with site wise unique identifier and it can generate set of html based on the HTML

It introduce two new remark node type

- IndexedBlock
- IndexedBlockChildren

## The flow

- remark-indexed-block: Generate new markdown with indexed block syntax
- mdast-utils-to-hast(remark-rehype): We transfer custom node type to hast, and then transfer to HTML
  - We could use [remark-rehype-options.handler](https://github.com/remarkjs/remark-rehype#optionshandlers) to handle custom node

## Normal Logic

1. It will use the file's relative file path and uuidv5 to form a unique id. For example, if you have a file's path is `/src/data/blog/hi-i-am-indexed-block`(relative to the root of the project.) It will tokenize this path and generate id.
2. It will use the first 6 character as the prefix.
3. The suffix is depend on the order of the section.

```md
# Header 1

## Sub Header 1

hi, I am a block!
```

Every unique representation will be a block, take above markdown for example, `Header 1`, `Sub Header 1` and the `hi, I am a block!` are three separated blocks. The suffix will be determined by their order. If the calculated prefix is r43erf the overall index of `Header 1` will be `r43erf-1`

## Worth notice Caveat

- Because remark is a esm module, we have to keep an eye one that:
  - We use `tsc` to build the package and imply that we are using esm by type config in package.json `type: "module"`
  - When testing the plugin, we run the script with `node`, but node is running with CommonJS by default, you need to imply that you want to run with esm by this flag `--experimental-specifier-resolution=node`
- unified.parse will generate a syntax tree, but does not run plugins/transformers.
  - ref: https://github.com/unifiedjs/unified/discussions/162
  - If you want unified to run plugin you could write something like this
  ```js
  const ast = unified().use(remarkParse).parse(buffer);
  const transformedAst = await unified().use(remarkIndexedBlock).run(ast);
  ```
- You could declare custom node type

  ```js
    type IndexedBlock = {
      type: "IndexedBlock";
      id: string;
      children: Content[];
    };

    declare module "mdast" {
      interface BlockContentMap {
        indexedBlock: IndexedBlock;
      }
    }
  ```
