---
title: "SQLite3 头文件注释翻译"
date: 2007-07-20T02:45:04+08:00
lastmod: 2007-07-20T02:45:04+08:00
draft: false
isCJKLanguage: true
---

的一部分……

因为头文件代码注释的非常好，可惜是英文的，虽然偶理解起来米问题，可是……

于是顺便练练手，不知道这种翻译是否表达了原文的准确意思……欢迎砸砖头……

翻译样本在下面:<!--more-->
/*
** CAPI3REF: One-Step Query Execution Interface
**
** This interface is used to do a one-time evaluatation of zero
** or more SQL statements.  UTF-8 text of the SQL statements to
** be evaluted is passed in as the second parameter.  The statements
** are prepared one by one using [sqlite3_prepare()], evaluated
** using [sqlite3_step()], then destroyed using [sqlite3_finalize()].
**
** If one or more of the SQL statements are queries, then
** the callback function specified by the 3rd parameter is
** invoked once for each row of the query result.  This callback
** should normally return 0.  If the callback returns a non-zero
** value then the query is aborted, all subsequent SQL statements
** are skipped and the sqlite3_exec() function returns the SQLITE_ABORT.
**
** The 4th parameter to this interface is an arbitrary pointer that is
** passed through to the callback function as its first parameter.
**
** The 2nd parameter to the callback function is the number of
** columns in the query result.  The 3rd parameter to the callback
** is an array of strings holding the values for each column
** as extracted using [sqlite3_column_text()].
** The 4th parameter to the callback is an array of strings
** obtained using [sqlite3_column_name()] and holding
** the names of each column.
**
** The callback function may be NULL, even for queries.  A NULL
** callback is not an error.  It just means that no callback
** will be invoked.
**
** If an error occurs while parsing or evaluating the SQL (but
** not while executing the callback) then an appropriate error
** message is written into memory obtained From&nbsp;[sqlite3_malloc()] and
** *errmsg is made to point to that message.  The calling function
** is responsible for freeing the memory that holds the error
** message.   Use [sqlite3_free()] for this.  If errmsg==NULL,
** then no error message is ever written.
**
** The return value is is SQLITE_OK if there are no errors and
** some other [SQLITE_OK | return code] if there is an error.
** The particular return value depends on the type of error.
**
*/
int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluted */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);

/*
** CAPI3REF: 执行单步查询接口
**
** 该接口用于一次性执行零个或者更多的SQL指令。被执行的SQL指
** 令应采用UTF-8编码并作为第二个参数传入。这些指令将经过
** [sqlite3_prepare()]调整，使用[sqlite3_step()]执行，最后
** 被[sqlite3_finalize()]销毁掉。
**
** 如果存在SQL查询指令，那么按照第三个参数定义的回调函数将被
** 每条返回记录触发。该函数正常情况下应当返回0值。如果该函数
** 返回非零值，查询将被中断，所有在其之后的查询将不会进行，
** 而 sqlite3_exec() 函数将会返回 SQLITE_ABORT 代码。
**
** 该接口的第四个参数是一个无法无天的指针(任意类型指针)，
** 它将作为回调函数的第一个参数传入。
**
** 回调函数的第二个参数是查询结果的列数。第三个参数是一个字符
** 串数组，里面保存了用[sqlite3_column_text()]处理后的文本摘录。
**
** 回调函数的第四个参数是一个用[sqlite3_column_name()]获取的列
** 名字符串数组。
**
** 回调函数在任何情况下都可以为空(NULL)。空回调函数不会出错，
** 它只是表示没有回调函数将被执行。
**
** 在解析和执行SQL指令中如果发生错误(但并不包括在回调函数中发生的
** 错误)，出错信息将会写入用[sqlite3_malloc()]获取的内存中，并让
** *errmsg 指针指向该信息，调用此接口的函数应当使用[sqlite3_free()]
** 释放该内存空间。如果 errmsg 为空(==NULL)，则没有出错信息被写入。
**
** 如果没有出错，返回值将是SQLITE_OK；如果出错则会对特定的错误类型
** 返回[SQLITE_OK | return code]的错误代码。
**
*/
int sqlite3_exec(
  sqlite3*,                                  /* 一个打开的数据库 */
  const char *sql,                           /* 要执行的SQL */
  int (*callback)(void*,int,char**,char**),  /* 回调函数 */
  void *,                                    /* 回调函数的第一个参数 */
  char **errmsg                              /* 出错消息 */
);