riot shell 
===========

命令行结构体
```
typedef int (*shell_command_handler_t)(int argc, char **argv);
typedef struct shell_command_t {
    const char *name;
    const char *desc;
    shell_command_handler_t handler;
} shell_command_t;
```

添加命令
```
int cmd_handler(int argc, char **argv);

static const shell_command_t commands[] = {
    {"hello", "print Hello RIOT", cmd_handler}
};

int cmd_handler(int argc, char **argv)
{
    (void) argc;
    (void) argv;
    printf("Hello RIOT! \n");
    return 0;
}

int main(void)
{
    ...
    char line_buf[SHELL_DEFAULT_BUFSIZE];
    shell_run(commands, line_buf, SHELL_DEFAULT_BUFSIZE);
    ...
}

```