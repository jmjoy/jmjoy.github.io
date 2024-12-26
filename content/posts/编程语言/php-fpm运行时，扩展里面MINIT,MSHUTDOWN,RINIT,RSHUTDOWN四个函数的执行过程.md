参考文章：<http://blog.sina.com.cn/s/blog_4d88b5b50102wu8i.html>

我的测试代码片段：

```c
#define FILENAME "/tmp/test"

static void append_file(char *s) {
	FILE *fp = fopen(/tmp/test, "a");
	fputs(s, fp);
	fclose(fp);
}

/* {{{ PHP_MINIT_FUNCTION
 */
PHP_MINIT_FUNCTION(test)
{
    append_file("MODULE_INIT\n");
    /* If you have INI entries, uncomment these lines
    REGISTER_INI_ENTRIES();
    */
    return SUCCESS;
}
/* }}} */

/* {{{ PHP_MSHUTDOWN_FUNCTION
 */
PHP_MSHUTDOWN_FUNCTION(test)
{
    append_file("MODULE_SHUTDOWN\n");
    /* uncomment this line if you have INI entries
    UNREGISTER_INI_ENTRIES();
    */
    return SUCCESS;
}
/* }}} */

/* Remove if there's nothing to do at request start */
/* {{{ PHP_RINIT_FUNCTION
 */
PHP_RINIT_FUNCTION(test)
{
    append_file("REQUEST_INIT\n");
#if defined(COMPILE_DL_FUCK) && defined(ZTS)
    ZEND_TSRMLS_CACHE_UPDATE();
#endif
    return SUCCESS;
}
/* }}} */

/* Remove if there's nothing to do at request end */
/* {{{ PHP_RSHUTDOWN_FUNCTION
 */
PHP_RSHUTDOWN_FUNCTION(test)
{
    append_file("REQUEST_SHUTDOWN\n");
    return SUCCESS;
}
/* }}} */
```

然后预先创建好`/tmp/test`，注意php-fpm的pool的用户如果不是root的话，一定要设置成a+w的权限啊，不然php-fpm就报错`SIGABRT`或者`SIGSEGV`，让人摸不着头脑。

直接打印东西来调试是不行的，因为`PHP_RINIT_FUNCTION`和`PHP_RSHUTDOWN_FUNCTION`会在pool的进程里面被调用，标准输出已经被劫持了吧。

