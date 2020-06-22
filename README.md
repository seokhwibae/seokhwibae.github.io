# main.js
'''
const http = require("http");
const fs = require("fs");
const url = require("url");
const qs = require("querystring");
const { log } = require("console");

function templateList(fileList) {
  let list = "<ul>";
  let i = 0;
  while (i < fileList.length) {
    list = list + `<li><a href="/?id=${fileList[i]}">${fileList[i]}</a>></li>`;
    i += 1;
  }
  list += "</ul>";
  return list;
}

function templateHtml(title, list, body, control) {
  return `
  <!doctype html>
  <html>
  <head>
    <title>WEB1 - ${title}</title>
    <meta charset="utf-8">
  </head>
  <body>
    <h1><a href="/">WEB</a></h1>
    ${list}
    ${control}
    ${body}
  </body>
  </html>
  `;
}
const app = http.createServer(function (request, response) {
  const _url = request.url;
  const queryData = url.parse(_url, true).query;
  const pathName = url.parse(_url, true).pathname;
  if (pathName === "/") {
    if (queryData.id === undefined) {
      fs.readdir("./data", "utf8", function (error, fileList) {
        let description = "hello, nodejs";
        let title = "welcome";
        const list = templateList(fileList);
        const template = templateHtml(title, list, `<h1>${title}</h2><span>${description}</span>`, `<a href="/create">create</a>`);
        response.writeHead(200);
        response.end(template);
      });
    } else {
      fs.readdir("./data", function (error, fileList) {
        fs.readFile(`data/${queryData.id}`, "utf8", function (err, description) {
          let title = queryData.id;
          const list = templateList(fileList);
          const template = templateHtml(
            title,
            list,
            `<h1>${title}</h2><span>${description}</span>`,
            `<a href="/create">create</a>
            <a href="/update?id=${title}">update</a>
            <a href="/delete?id=${title}">delete</a>`
          );
          response.writeHead(200);
          response.end(template);
        });
      });
    }
  } else if (pathName === "/update") {
    fs.readdir("./data", function (error, fileList) {
      fs.readFile(`data/${queryData.id}`, "utf8", function (err, description) {
        let title = queryData.id;
        const list = templateList(fileList);
        const template = templateHtml(
          title,
          list,
          `<form action="/update_process" method="POST">
          <input type="hidden" value="${title}" name="id"/>
          <p>
            <input placeholder="이름" type="text" name="title" value="${title}" />
          </p>
          <p>
            <textarea placeholder="내용" name="description" cols="30" rows="10">${description}</textarea>
          </p>
          <p>
            <input type="submit" value="글쓰기" />
          </p>
        </form>`,
          ""
        );
        response.writeHead(200);
        response.end(template);
      });
    });
  } else if (pathName === "/update_process") {
    let body = "";
    request.on("data", function (data) {
      body += data;
    });
    request.on("end", function () {
      let post = qs.parse(body);
      let title = post.title;
      let description = post.description;
      let id = post.id;
      fs.rename(`data/${id}`, `data/${title}`, function (error) {});
      fs.writeFile(`data/${id}`, description, "utf8", function (err) {
        response.writeHead(302, { Location: `/?id=${qs.escape(title)}` });
        response.end();
      });
    });
  } else if (pathName === "/create") {
    fs.readdir("./data", function (error, fileList) {
      let title = "WEB - create";
      const list = templateList(fileList);
      const template = templateHtml(
        title,
        list,
        `<form action="/create_process" method="POST">
          <p>
            <input placeholder="이름" type="text" name="title" />
          </p>
          <p>
            <textarea placeholder="내용" name="description" cols="30" rows="10"></textarea>
          </p>
          <p>
            <input type="submit" value="글쓰기" />
          </p>
        </form>`,
        ""
      );
      response.writeHead(200);
      response.end(template);
    });
  } else if (pathName === "/create_process") {
    let body = "";
    request.on("data", function (data) {
      body += data;
    });
    request.on("end", function () {
      let post = qs.parse(body);
      let title = post.title;
      let description = post.description;
      fs.writeFile(`data/${title}`, description, "utf8", function (err) {
        response.writeHead(302, { Location: `/?id=${qs.escape(title)}` });
        response.end();
      });
    });
  } else if (pathName === "/delete") {
    fs.readdir("./data", function (error, fileList) {
      fs.readFile(`data/${queryData.id}`, "utf8", function (err, description) {
        let title = queryData.id;
        console.log(title);
        fs.unlink(`./data/${title}`, function (err) {
          if (err) throw err;

          console.log("file deleted");
        });

        response.writeHead(302, { Location: `/` });
        response.end();
      });
    });
  } else {
    response.writeHead(404);
    response.end("Not found");
  }
});
app.listen(3000);
'''
