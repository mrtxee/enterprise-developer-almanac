---
aliases:
---
# enterprise-developer-almanac
> альманах корпоративного разработчика

---





# toc-1

```dataviewjs


const files = dv.pages('"/"').sort(p => p.file.path);
const groups = {};

for (let file of files) {
    const parts = file.file.folder.split('/');
    let current = groups;
    
    for (let part of parts) {
        if (!current[part]) current[part] = {};
        current = current[part];
    }
    current[file.file.name] = true;
}

function print(obj, level = 0) {
    for (let key in obj) {
        if (key === '_') continue;
        const isFolder = Object.keys(obj[key]).length > 0;
        dv.span('  '.repeat(level) + (isFolder ? '📁 ' : '📄 ') + key);
        dv.paragraph('');
        if (isFolder) print(obj[key], level + 1);
    }
}

print(groups);

```














