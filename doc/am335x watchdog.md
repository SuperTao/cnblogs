am335x watchdog

内核文档kernel/Documentation/watchdog

Qt@aplex:~/kernel/7109/linux-3.2.0/Documentation/watchdog$ ll
total 88
drwxrwxr-x  3 Qt Qt  4096 Jun  8 15:11 ./
drwxrwxr-x 94 Qt Qt 12288 Apr 28 13:09 ../
-rwxrwxr-x  1 Qt Qt   576 Nov 20  2013 00-INDEX*
-rwxrwxr-x  1 Qt Qt  6789 Nov 20  2013 convert_drivers_to_kernel_api.txt*
-rwxrwxr-x  1 Qt Qt  3989 Nov 20  2013 hpwdt.txt*
-rwxrwxr-x  1 Qt Qt  2266 Nov 20  2013 pcwd-watchdog.txt*
drwxrwxr-x  2 Qt Qt  4096 Jun  8 15:11 src/
-rwxrwxr-x  1 Qt Qt  8442 Nov 20  2013 watchdog-api.txt*
-rwxrwxr-x  1 Qt Qt  8387 Nov 20  2013 watchdog-kernel-api.txt*
-rwxrwxr-x  1 Qt Qt 16450 Nov 20  2013 watchdog-parameters.txt*
-rwxrwxr-x  1 Qt Qt  1825 Nov 20  2013 wdt.txt*


参考程序
内核文档kernel/Documentation/watchdog/src/watchdog-simple.c
内核文档kernel/Documentation/watchdog/src/watchdog-test.c
```
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/types.h>
#include <linux/watchdog.h>

int fd;

/*
 * This function simply sends an IOCTL to the driver, which in turn ticks
 * the PC Watchdog card to reset its internal timer so it doesn't trigger
 * a computer reset.
 */
static void keep_alive(void)
{
    int dummy;

    ioctl(fd, WDIOC_KEEPALIVE, &dummy);
}

/*
 * The main program.  Run the program with "-d" to disable the card,
 * or "-e" to enable the card.
 */
int main(int argc, char *argv[])
{
    int flags;

    fd = open("/dev/watchdog", O_WRONLY);

    if (fd == -1) {
    fprintf(stderr, "Watchdog device not enabled.\n");
    fflush(stderr);
    exit(-1);
    }

    if (argc > 1) {
    if (!strncasecmp(argv[1], "-d", 2)) {
        flags = WDIOS_DISABLECARD;
        ioctl(fd, WDIOC_SETOPTIONS, &flags);
        fprintf(stderr, "Watchdog card disabled.\n");
        fflush(stderr);
        exit(0);
    } else if (!strncasecmp(argv[1], "-e", 2)) {
        flags = WDIOS_ENABLECARD;
        ioctl(fd, WDIOC_SETOPTIONS, &flags);
        fprintf(stderr, "Watchdog card enabled.\n");
        fflush(stderr);
        exit(0);
    } else {
        fprintf(stderr, "-d to disable, -e to enable.\n");
        fprintf(stderr, "run by itself to tick the card.\n");
        fflush(stderr);
        exit(0);
    }
    } else {
    fprintf(stderr, "Watchdog Ticking Away!\n");
    fflush(stderr);
    }

    while(1) {
    keep_alive();
    sleep(1);
    }
}
```