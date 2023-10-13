import { jest } from "@jest/globals";
import axios from "axios";
import MockAdapter from "axios-mock-adapter";
import api from "../api/index.js";
import { calculateRank } from "../src/calculateRank.js";
import { renderStatsCard } from "../src/cards/stats-card.js";
import { CONSTANTS, renderError } from "../src/common/utils.js";
import { expect, it, describe, afterEach } from "@jest/globals";

const stats = {
  name: "Anurag Hazra",
  totalStars: 100,
  totalCommits: 200,
  totalIssues: 300,
  totalPRs: 400,
  totalPRsMerged: 320,
  mergedPRsPercentage: 80,
  totalReviews: 50,
  totalDiscussionsStarted: 10,
  totalDiscussionsAnswered: 40,
  contributedTo: 50,
  rank: null,
};

stats.rank = calculateRank({
  all_commits: false,
  commits: stats.totalCommits,
  prs: stats.totalPRs,
  reviews: stats.totalReviews,
  issues: stats.totalIssues,
  repos: 1,
  stars: stats.totalStars,
  followers: 0,
});

const data_stats = {
  data: {
    user: {
      name: stats.name,
      repositoriesContributedTo: { totalCount: stats.contributedTo },
      contributionsCollection: {
        totalCommitContributions: stats.totalCommits,
        totalPullRequestReviewContributions: stats.totalReviews,
      },
      pullRequests: { totalCount: stats.totalPRs },
      mergedPullRequests: { totalCount: stats.totalPRsMerged },
      openIssues: { totalCount: stats.totalIssues },
      closedIssues: { totalCount: 0 },
      followers: { totalCount: 0 },
      repositoryDiscussions: { totalCount: stats.totalDiscussionsStarted },
      repositoryDiscussionComments: {
        totalCount: stats.totalDiscussionsAnswered,
      },
      repositories: {
        totalCount: 1,
        nodes: [{ stargazers: { totalCount: 100 } }],
        pageInfo: {
          hasNextPage: false,
          endCursor: "cursor",
        },
      },
    },
  },
};

const error = {
  errors: [
    {
      type: "NOT_FOUND",
      path: ["user"],
      locations: [],
      message: "Could not fetch user",
    },
  ],
};

const mock = new MockAdapter(axios);

const faker = (query, data) => {
  const req = {
    query: {
      username: "anuraghazra",
      ...query,
    },
  };
  const res = {
    setHeader: jest.fn(),
    send: jest.fn(),
  };
  mock.onPost("https://api.github.com/graphql").replyOnce(200, data);

  return { req, res };
};

afterEach(() => {
  mock.reset();
});

describe("Test /api/", () => {
  it("should test the request", async () => {
    const { req, res } = faker({}, data_stats);

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(renderStatsCard(stats, { ...req.query }));
  });

  it("should render error card on error", async () => {
    const { req, res } = faker({}, error);

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(
      renderError(
        error.errors[0].message,
        "Make sure the provided username is not an organization",
      ),
    );
  });

  it("should get the query options", async () => {
    const { req, res } = faker(
      {
        username: "anuraghazra",
        hide: "issues,prs,contribs",
        show_icons: true,
        hide_border: true,
        line_height: 100,
        title_color: "fff",
        icon_color: "fff",
        text_color: "fff",
        bg_color: "fff",
      },
      data_stats,
    );

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(
      renderStatsCard(stats, {
        hide: ["issues", "prs", "contribs"],
        show_icons: true,
        hide_border: true,
        line_height: 100,
        title_color: "fff",
        icon_color: "fff",
        text_color: "fff",
        bg_color: "fff",
      }),
    );
  });

  it("should have proper cache", async () => {
    const { req, res } = faker({}, data_stats);

    await api(req, res);

    expect(res.setHeader.mock.calls).toEqual([
      ["Content-Type", "image/svg+xml"],
      [
        "Cache-Control",
        `max-age=${CONSTANTS.SIX_HOURS / 2}, s-maxage=${
          CONSTANTS.SIX_HOURS
        }, stale-while-revalidate=${CONSTANTS.ONE_DAY}`,
      ],
    ]);
  });

  it("should set proper cache", async () => {
    const cache_seconds = 35000;
    const { req, res } = faker({ cache_seconds }, data_stats);
    await api(req, res);

    expect(res.setHeader.mock.calls).toEqual([
      ["Content-Type", "image/svg+xml"],
      [
        "Cache-Control",
        `max-age=${
          cache_seconds / 2
        }, s-maxage=${cache_seconds}, stale-while-revalidate=${
          CONSTANTS.ONE_DAY
        }`,
      ],
    ]);
  });

  it("should set shorter cache when error", async () => {
    const { req, res } = faker({}, error);
    await api(req, res);

    expect(res.setHeader.mock.calls).toEqual([
      ["Content-Type", "image/svg+xml"],
      [
        "Cache-Control",
        `max-age=${CONSTANTS.ERROR_CACHE_SECONDS / 2}, s-maxage=${
          CONSTANTS.ERROR_CACHE_SECONDS
        }, stale-while-revalidate=${CONSTANTS.ONE_DAY}`,
      ],
    ]);
  });

  it("should set proper cache with clamped values", async () => {
    {
      let { req, res } = faker({ cache_seconds: 200000 }, data_stats);
      await api(req, res);

      expect(res.setHeader.mock.calls).toEqual([
        ["Content-Type", "image/svg+xml"],
        [
          "Cache-Control",
          `max-age=${CONSTANTS.ONE_DAY / 2}, s-maxage=${
            CONSTANTS.ONE_DAY
          }, stale-while-revalidate=${CONSTANTS.ONE_DAY}`,
        ],
      ]);
    }

    // note i'm using block scoped vars
    {
      let { req, res } = faker({ cache_seconds: 0 }, data_stats);
      await api(req, res);

      expect(res.setHeader.mock.calls).toEqual([
        ["Content-Type", "image/svg+xml"],
        [
          "Cache-Control",
          `max-age=${CONSTANTS.SIX_HOURS / 2}, s-maxage=${
            CONSTANTS.SIX_HOURS
          }, stale-while-revalidate=${CONSTANTS.ONE_DAY}`,
        ],
      ]);
    }

    {
      let { req, res } = faker({ cache_seconds: -10000 }, data_stats);
      await api(req, res);

      expect(res.setHeader.mock.calls).toEqual([
        ["Content-Type", "image/svg+xml"],
        [
          "Cache-Control",
          `max-age=${CONSTANTS.SIX_HOURS / 2}, s-maxage=${
            CONSTANTS.SIX_HOURS
          }, stale-while-revalidate=${CONSTANTS.ONE_DAY}`,
        ],
      ]);
    }
  });

  it("should allow changing ring_color", async () => {
    const { req, res } = faker(
      {
        username: "anuraghazra",
        hide: "issues,prs,contribs",
        show_icons: true,
        hide_border: true,
        line_height: 100,
        title_color: "fff",
        ring_color: "0000ff",
        icon_color: "fff",
        text_color: "fff",
        bg_color: "fff",
      },
      data_stats,
    );

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(
      renderStatsCard(stats, {
        hide: ["issues", "prs", "contribs"],
        show_icons: true,
        hide_border: true,
        line_height: 100,
        title_color: "fff",
        ring_color: "0000ff",
        icon_color: "fff",
        text_color: "fff",
        bg_color: "fff",
      }),
    );
  });

  it("should render error card if username in blacklist", async () => {
    const { req, res } = faker({ username: "renovate-bot" }, data_stats);

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(renderError("Something went wrong"));
  });

  it("should render error card when wrong locale is provided", async () => {
    const { req, res } = faker({ locale: "asdf" }, data_stats);

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(
      renderError("Something went wrong", "Language not found"),
    );
  });

  it("should render error card when include_all_commits true and upstream API fails", async () => {
    mock
      .onGet("https://api.github.com/search/commits?q=author:anuraghazra")
      .reply(200, { error: "Some test error message" });

    const { req, res } = faker(
      { username: "anuraghazra", include_all_commits: true },
      data_stats,
    );

    await api(req, res);

    expect(res.setHeader).toBeCalledWith("Content-Type", "image/svg+xml");
    expect(res.send).toBeCalledWith(
      renderError("Could not fetch total commits.", "Please try again later"),
    );
  });
});
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


