import fs from "fs";
import { themes } from "../themes/index.js";

const TARGET_FILE = "./themes/README.md";
const REPO_CARD_LINKS_FLAG = "<!-- REPO_CARD_LINKS -->";
const STAT_CARD_LINKS_FLAG = "<!-- STATS_CARD_LINKS -->";

const STAT_CARD_TABLE_FLAG = "<!-- STATS_CARD_TABLE -->";
const REPO_CARD_TABLE_FLAG = "<!-- REPO_CARD_TABLE -->";

const THEME_TEMPLATE = `## Available Themes

<!-- DO NOT EDIT THIS FILE DIRECTLY -->

With inbuilt themes, you can customize the look of the card without doing any manual customization.

Use \`?theme=THEME_NAME\` parameter like so :-

\`\`\`md
![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=anuraghazra&theme=dark&show_icons=true)
\`\`\`

## Stats

> These themes work both for the Stats Card and Repo Card.

| | | |
| :--: | :--: | :--: |
${STAT_CARD_TABLE_FLAG}

## Repo Card

> These themes work both for the Stats Card and Repo Card.

| | | |
| :--: | :--: | :--: |
${REPO_CARD_TABLE_FLAG}

${STAT_CARD_LINKS_FLAG}

${REPO_CARD_LINKS_FLAG}


[add-theme]: https://github.com/anuraghazra/github-readme-stats/edit/master/themes/index.js

Want to add a new theme? Consider reading the [contribution guidelines](../CONTRIBUTING.md#themes-contribution) :D
`;

const createRepoMdLink = (theme) => {
  return `\n[${theme}_repo]: https://github-readme-stats.vercel.app/api/pin/?username=anuraghazra&repo=github-readme-stats&cache_seconds=86400&theme=${theme}`;
};
const createStatMdLink = (theme) => {
  return `\n[${theme}]: https://github-readme-stats.vercel.app/api?username=anuraghazra&show_icons=true&hide=contribs,prs&cache_seconds=86400&theme=${theme}`;
};

const generateLinks = (fn) => {
  return Object.keys(themes)
    .map((name) => fn(name))
    .join("");
};

const createTableItem = ({ link, label, isRepoCard }) => {
  if (!link || !label) {
    return "";
  }
  return `\`${label}\` ![${link}][${link}${isRepoCard ? "_repo" : ""}]`;
};

const generateTable = ({ isRepoCard }) => {
  const rows = [];
  const themesFiltered = Object.keys(themes).filter(
    (name) => name !== (!isRepoCard ? "default_repocard" : "default"),
  );

  for (let i = 0; i < themesFiltered.length; i += 3) {
    const one = themesFiltered[i];
    const two = themesFiltered[i + 1];
    const three = themesFiltered[i + 2];

    let tableItem1 = createTableItem({ link: one, label: one, isRepoCard });
    let tableItem2 = createTableItem({ link: two, label: two, isRepoCard });
    let tableItem3 = createTableItem({ link: three, label: three, isRepoCard });

    if (three === undefined) {
      tableItem3 = `[Add your theme][add-theme]`;
    }
    rows.push(`| ${tableItem1} | ${tableItem2} | ${tableItem3} |`);

    // if it's the last row & the row has no empty space push a new row
    if (three && i + 3 === themesFiltered.length) {
      rows.push(`| [Add your theme][add-theme] | | |`);
    }
  }

  return rows.join("\n");
};

const buildReadme = () => {
  return THEME_TEMPLATE.split("\n")
    .map((line) => {
      if (line.includes(REPO_CARD_LINKS_FLAG)) {
        return generateLinks(createRepoMdLink);
      }
      if (line.includes(STAT_CARD_LINKS_FLAG)) {
        return generateLinks(createStatMdLink);
      }
      if (line.includes(REPO_CARD_TABLE_FLAG)) {
        return generateTable({ isRepoCard: true });
      }
      if (line.includes(STAT_CARD_TABLE_FLAG)) {
        return generateTable({ isRepoCard: false });
      }
      return line;
    })
    .join("\n");
};

fs.writeFileSync(TARGET_FILE, buildReadme());
### 📌 Introduction
 
Hello there! 👋 I'm Ryan Ribeiro, a relatively new member of the GitHub community. As a Computer Engineering student from Goiânia, Goiás State of Brazil, I'm constantly honing my skills and exploring new technologies. While I primarily program in C, I'm also diving into the world of Java object-oriented programming.

### 👨‍💻 Skills & Interests

Take a look at my highlighted skills and interests:

- Programming languages: C, Java
- Learning about Object-Oriented Programming in Java

### 💼 Projects
Here are some of my projects. Feel free to check them out!

- [Final-Project-C-Language-Sales-Software](https://github.com/ryan-ribeiro/Final-Project-C-Language-Sales-Software) - My final project was made in C language about a pizza company

- [iec2023_1_AeB](https://github.com/ryan-ribeiro/iec2023_1_AeB): This repository contains a collection of introduction projects developed through my college classes, using Python, HTML/CSS, and JavaScript.

- [c-language](https://github.com/ryan-ribeiro/c-language): In this repository, you can find various projects that I have implemented in C language. These projects demonstrate my proficiency in C and my ability to solve different programming challenges.

- [poo-2023-02](https://github.com/ryan-ribeiro/poo-2023-02): This repository contains my projects related to Java and Object-Oriented Programming. It represents my understanding of Java concepts and my ability to create well-designed and maintainable applications.


### 📞 Contact

Feel free to reach out to me! You can contact me through the following channels:

- Email: [ryan.ribeiro@discente.ufg.br](mailto:ryan.ribeiro@discente.ufg.br)
- LinkedIn: [Ryan Ribeiro](https://www.linkedin.com/in/ryan-ribeiro)

### 📊 GitHub Stats

![GitHub Stats](https://github-readme-stats.vercel.app/api?username=ryan-ribeiro)

Here's a glimpse into my GitHub stats. While I might be a relative newcomer, I'm excited to contribute more and expand my presence on GitHub.


### 🌟 Popular Projects, Badges, and More!

[![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=ryan-ribeiro&layout=compact)](https://github.com/ryan-ribeiro)


