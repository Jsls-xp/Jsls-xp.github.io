## 一、Linux中概念
### 1.1 Uid
Linux 是一个多用户操作系统，系统中可以同时存在有多个用户。每个用户有一个用户名，也就是登录时输入的的用户名。用户名在系统中会对应一个整数值 UID，是用户在系统中的唯一标识。

### 1.2 Gid
为了方便管理系统中的多个用户，系统将用户进行了分组，每个用户可以在一个或者多个组中。每个组也有自己的组名和组 id（gid）。一个用户可以同时在多个组中。

### 1.3 Pid
系统在程序运行时，会为每个可执行程序分配一个唯一的进程ID（PID），PID的直接作用是为了表明该程序所拥有的文件操作权限，不同的可执行程序运行时互不影响，相互之间的数据访问具有权限限制。

## 二、Android中的Uid、UserId、AppId
frameworks/base/core/java/android/os/UserHandle.java提供了计算方法
```java
public final class UserHandle implements Parcelable {
    /**
     * @hide Range of uids allocated for a user.
     */
    @UnsupportedAppUsage
    public static final int PER_USER_RANGE = 100000;

    /**
     * @hide A user id constant to indicate the "owner" user of the device
     * @deprecated Consider using either {@link UserHandle#USER_SYSTEM} constant or
     * check the target user's flag {@link android.content.pm.UserInfo#isAdmin}.
     */
    @UnsupportedAppUsage
    @Deprecated
    public static final @UserIdInt int USER_OWNER = 0;

    /**
     * @hide A user handle to indicate the primary/owner user of the device
     * @deprecated Consider using either {@link UserHandle#SYSTEM} constant or
     * check the target user's flag {@link android.content.pm.UserInfo#isAdmin}.
     */
    @UnsupportedAppUsage
    @Deprecated
    public static final @NonNull UserHandle OWNER = new UserHandle(USER_OWNER);

    /** @hide A user id constant to indicate the "system" user of the device */
    @UnsupportedAppUsage
    @TestApi
    public static final @UserIdInt int USER_SYSTEM = 0;

    /** @hide A user serial constant to indicate the "system" user of the device */
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
    public static final int USER_SERIAL_SYSTEM = 0;

    /** @hide A user handle to indicate the "system" user of the device */
    @SystemApi
    public static final @NonNull UserHandle SYSTEM = new UserHandle(USER_SYSTEM);

    /**
     * @hide Enable multi-user related side effects. Set this to false if
     * there are problems with single user use-cases.
     */
    @UnsupportedAppUsage
    public static final boolean MU_ENABLED = true;

    /** The userId represented by this UserHandle. */
    @UnsupportedAppUsage
    final @UserIdInt int mHandle;

    /**
     * Checks to see if the user id is the same for the two uids, i.e., they belong to the same
     * user.
     * @hide
     */
    public static boolean isSameUser(int uid1, int uid2) {
        return getUserId(uid1) == getUserId(uid2);
    }

    /**
     * Checks to see if both uids are referring to the same app id, ignoring the user id part of the
     * uids.
     * @param uid1 uid to compare
     * @param uid2 other uid to compare
     * @return whether the appId is the same for both uids
     * @hide
     */
    @UnsupportedAppUsage
    public static boolean isSameApp(int uid1, int uid2) {
        return getAppId(uid1) == getAppId(uid2);
    }

     /**
     * Whether a UID belongs to a regular app. *Note* "Not a regular app" does not mean
     * "it's system", because of isolated UIDs. Use {@link #isCore} for that.
     * @hide
     */
    @UnsupportedAppUsage
    @TestApi
    public static boolean isApp(int uid) {
        if (uid > 0) {
            final int appId = getAppId(uid);
            return appId >= Process.FIRST_APPLICATION_UID && appId <= Process.LAST_APPLICATION_UID;
        } else {
            return false;
        }
    }

    /**
     * Whether a UID belongs to a system core component or not.
     * @hide
     */
    public static boolean isCore(int uid) {
        if (uid >= 0) {
            final int appId = getAppId(uid);
            return appId < Process.FIRST_APPLICATION_UID;
        } else {
            return false;
        }
    }

    /**
     * Returns the user for a given uid.
     * @param uid A uid for an application running in a particular user.
     * @return A {@link UserHandle} for that user.
     */
    public static UserHandle getUserHandleForUid(int uid) {
        return of(getUserId(uid));
    }

    /**
     * Returns the user id for a given uid.
     * @hide
     */
    @UnsupportedAppUsage
    @TestApi
    public static @UserIdInt int getUserId(int uid) {
        if (MU_ENABLED) {
            return uid / PER_USER_RANGE;
        } else {
            return UserHandle.USER_SYSTEM;
        }
    }

    /** @hide */
    @UnsupportedAppUsage
    public static @UserIdInt int getCallingUserId() {
        return getUserId(Binder.getCallingUid());
    }

    /** @hide */
    public static @AppIdInt int getCallingAppId() {
        return getAppId(Binder.getCallingUid());
    }   

    /** @hide */
    @VisibleForTesting
    public static UserHandle getUserHandleFromExtraCache(@UserIdInt int userId) {
        synchronized (sExtraUserHandleCache) {
            final UserHandle extraCached = sExtraUserHandleCache.get(userId);
            if (extraCached != null) {
                return extraCached;
            }
            if (sExtraUserHandleCache.size() >= MAX_EXTRA_USER_HANDLE_CACHE_SIZE) {
                sExtraUserHandleCache.removeAt(
                        (new Random()).nextInt(MAX_EXTRA_USER_HANDLE_CACHE_SIZE));
            }
            final UserHandle newHandle = new UserHandle(userId);
            sExtraUserHandleCache.put(userId, newHandle);
            return newHandle;
        }
    }

    /**
     * Returns the uid that is composed from the userId and the appId.
     * @hide
     */
    @UnsupportedAppUsage
    @TestApi
    public static int getUid(@UserIdInt int userId, @AppIdInt int appId) {
        if (MU_ENABLED && appId >= 0) {
            return userId * PER_USER_RANGE + (appId % PER_USER_RANGE);
        } else {
            return appId;
        }
    }

    /**
     * Returns the uid representing the given appId for this UserHandle.
     *
     * @param appId the AppId to compose the uid
     * @return the uid representing the given appId for this UserHandle
     * @hide
     */
    @SystemApi
    public int getUid(@AppIdInt int appId) {
        return getUid(getIdentifier(), appId);
    }

    /**
     * Returns the app id (or base uid) for a given uid, stripping out the user id from it.
     * @hide
     */
    @SystemApi
    public static @AppIdInt int getAppId(int uid) {
        return uid % PER_USER_RANGE;
    }

    /**
     * Returns the user id of the current process
     * @return user id of the current process
     * @hide
     */
    @SystemApi
    public static @UserIdInt int myUserId() {
        return getUserId(Process.myUid());
    }

    /**
     * Returns true if this UserHandle refers to the owner user; false otherwise.
     * @return true if this UserHandle refers to the owner user; false otherwise.
     * @hide
     * @deprecated please use {@link #isSystem()} or check for
     * {@link android.content.pm.UserInfo#isPrimary()}
     * {@link android.content.pm.UserInfo#isAdmin()} based on your particular use case.
     */
    @Deprecated
    @SystemApi
    public boolean isOwner() {
        return this.equals(OWNER);
    }

    /**
     * @return true if this UserHandle refers to the system user; false otherwise.
     * @hide
     */
    @SystemApi
    public boolean isSystem() {
        return this.equals(SYSTEM);
    }

    /** @hide */
    @UnsupportedAppUsage
    public UserHandle(@UserIdInt int userId) {
        mHandle = userId;
    }

    /**
     * Returns the userId stored in this UserHandle.
     * @hide
     */
    @SystemApi
    public @UserIdInt int getIdentifier() {
        return mHandle;
    }

}
```
### 2.1 计算uid
```java
public static int getUid(@UserIdInt int userId, @AppIdInt int appId) {
    if (MU_ENABLED && appId >= 0) {
        return userId * PER_USER_RANGE + (appId % PER_USER_RANGE);
    } else {
        return appId;
    }
}
```
uid = userId * PER_USER_RANGE + (appId % PER_USER_RANGE)  
    = userId * 100000 + appId % 100000
可通过Process.myUid()获取

### 2.2 通过uid计算userId
```java
public static @UserIdInt int getUserId(int uid) {
    if (MU_ENABLED) {
        return uid / PER_USER_RANGE;
    } else {
        return UserHandle.USER_SYSTEM;
    }
}
```
userId = uid / PER_USER_RANGE = uid / 100000

### 2.3 通过uid计算appId  
```java
public static @AppIdInt int getAppId(int uid) {
    return uid % PER_USER_RANGE;
}
```
appId = uid % PER_USER_RANGE = uid % 100000

### 2.4 adb shell ps命令
```java
adb shell ps -A

USER           PID  PPID        VSZ    RSS WCHAN            ADDR S NAME                       
root             1     0   11131956  12064 0                   0 R init
root             2     0          0      0 kthreadd            0 S [kthreadd]
root             3     2          0      0 kthread_worker_fn   0 S [pool_workqueue_release]
u0_a285      23876  1235   15468748 216584 do_epoll_wait       0 S com.example.testandroid.v
u999_a285    24575  1235   15119612 213288 do_epoll_wait       0 S com.example.testandroid.v
```
```java
adb shell ps -T

USER           PID   TID  PPID        VSZ    RSS WCHAN            ADDR S CMD            
root             1     1     0   11131956  11936 do_epoll_wait       0 S init
root             1   312     0   11131956  11936 do_epoll_wait       0 S init
root             1   313     0   11131956  11936 do_epoll_wait       0 S init
root             2     2     0          0      0 kthreadd            0 S kthreadd
root             3     3     2          0      0 kthread_worker_fn   0 S pool_workqueue_release
u0_a285      23876 23876  1235   15163260 215404 do_freezer_trap     0 S e.testandroid.v
u0_a285      23876 32265  1235   15163260 215404 do_freezer_trap     0 S Signal Catcher
u999_a285    24575 24575  1235   15119612 183460 do_epoll_wait       0 S e.testandroid.v
u999_a285    24575 32384  1235   15119612 183460 do_sigtimedwait     0 S Signal Catcher
```
```
字段        含义       
USER    所有者用户的用户名，表示运行进程的用户。	
PID     进程ID（Process ID），唯一标识每个进程的数字。	
PPID	父进程ID（Parent Process ID），标识创建当前进程的父进程的ID。	
VSZ	    虚拟内存大小（Virtual Memory Size），进程当前使用的虚拟内存大小（以KB为单位）。
RSS	    常驻集大小（Resident Set Size），表示进程占用的物理内存大小（以KB为单位）。	
WCHAN	当前等待的内核函数（The name of the kernel function in which the process is sleeping）。	
ADDR	内存地址（Memory Address），进程的内存地址。	
S	    进程状态，表示进程的当前状态，此处为"S"表示休眠（sleeping）状态。
NAME	命令名（Command Name），表示进程的名称或命令。
TID     线程ID(Thread ID)
```

### 2.5 几种UID
```java
/**
 * Tools for managing OS processes.
 */
public class Process {
    /**
     * An invalid UID value.
     */
    public static final int INVALID_UID = -1;

    /**
     * Defines the root UID.
     */
    public static final int ROOT_UID = 0;

    /**
     * Defines the UID/GID under which system code runs.
     */
    public static final int SYSTEM_UID = 1000;

    /**
     * Defines the UID/GID under which the telephony code runs.
     */
    public static final int PHONE_UID = 1001;

    /**
     * Defines the UID/GID for the user shell.
     */
    public static final int SHELL_UID = 2000;

    /**
     * Defines the start of a range of UIDs (and GIDs), going from this
     * number to {@link #LAST_APPLICATION_UID} that are reserved for assigning
     * to applications.
     */
    public static final int FIRST_APPLICATION_UID = 10000;

    /**
     * Last of application-specific UIDs starting at
     * {@link #FIRST_APPLICATION_UID}.
     */
    public static final int LAST_APPLICATION_UID = 19999;
}
```

## 三、Android中UID:u0_axx的由来
Android中源码中:
bionic/libc/bionic/grp_pwd.cpp
```java
/*please check pointer taskinfo NOT NULL before call*/
int mc_get_task_info(int pid, int tid, struct task_info* taskinfo)
{
    int fd, rsize = 0;

    char proc_path[60];
    memset(proc_path, 0, sizeof(proc_path));
    sprintf(proc_path, "/proc/%d", pid);
    struct stat st;
    if ( stat(proc_path, &st) != 0) {
        fprintf(stderr, "%d not exist.\n", pid);
        return -1;
    }
    //user for task
    struct passwd *pw;
    pw = getpwuid(st.st_uid);
    if(pw == 0) {
        sprintf(taskinfo->user,"%d",(int)st.st_uid);
    } else {
        strcpy(taskinfo->user,pw->pw_name);
    }

    taskinfo->pid = (tid == 0) ? pid : tid;

    //cmdline for task name
    sprintf(proc_path, "/proc/%d/task/%d/cmdline", pid, taskinfo->pid);
    fd = open(proc_path, O_RDONLY);
    if (fd < 0) {
        fprintf(stderr, "open cmdline error:%s\n", strerror(errno));
        return -1;
    }
    rsize = read(fd, (taskinfo->u).cmdline, 127);
    if (rsize < 0) {
        fprintf(stderr, "read cmdline error:%s\n", strerror(errno));
        close(fd);
        return -1;
    }
    (taskinfo->u).cmdline[rsize] = 0;  //rsize: here also means real read counts
    close(fd);

    //stat for taskinfo(w/ thread name)
    memset(proc_path, 0, sizeof(proc_path));
    sprintf(proc_path, "/proc/%d/task/%d/stat", pid, (tid == 0) ? pid : tid);
    fd = open(proc_path, O_RDONLY);
    if (fd < 0) {
        fprintf(stderr, "open task stat error:%s\n", strerror(errno));
        return -1;
    }
    char* stat_content = (char*)malloc(1024);
    memset(stat_content, 0, 1024);
    rsize = read(fd, stat_content, 1023);
    if (rsize < 0) {
        fprintf(stderr, "read task stat error:%s\n", strerror(errno));
        return -1;
    }
    *(stat_content + rsize) = 0;  //rsize: here also means real read counts
    //fprintf(stderr, "%s\n", stat_content);
    close(fd);

    char* s = stat_content, *e;
    s = strchr(s, ' ');        // skip pid
    s = s + 2;                 // skip " ("
    e = strchr(s, ')');
    if ((taskinfo->u).cmdline[0] == 0) {
        strncpy((taskinfo->u).name, s, e-s);
    }

    s = strchr(s, ' ');                   //pass comm
    s = s + 1;                            //skip " "
    strncpy(taskinfo->state, s, 1);       //state: one char from "RSDZTW"
    s = strchr(s, ' ');                   //pass state
    s = s + 1;                            //skip " "

    if (s <= 1) {
        fprintf(stderr, "not valid stat content: \n%s\n", stat_content);
        return -1;
    }

    /*
    format at: /kernel/fs/proc/array.c
    seq_printf(m, "%d (%s) %c", pid_nr_ns(pid, ns), tcomm, state);
    seq_put_decimal_ll(m, ' ', ppid);
    seq_put_decimal_ll(m, ' ', pgid);
    seq_put_decimal_ll(m, ' ', sid);
    seq_put_decimal_ll(m, ' ', tty_nr);
    seq_put_decimal_ll(m, ' ', tty_pgrp);
    seq_put_decimal_ull(m, ' ', task->flags);
    seq_put_decimal_ull(m, ' ', min_flt);
    seq_put_decimal_ull(m, ' ', cmin_flt);
    seq_put_decimal_ull(m, ' ', maj_flt);
    seq_put_decimal_ull(m, ' ', cmaj_flt);
    seq_put_decimal_ull(m, ' ', cputime_to_clock_t(utime));
    seq_put_decimal_ull(m, ' ', cputime_to_clock_t(stime));
    seq_put_decimal_ll(m, ' ', cputime_to_clock_t(cutime));
    seq_put_decimal_ll(m, ' ', cputime_to_clock_t(cstime));
    seq_put_decimal_ll(m, ' ', priority);
    seq_put_decimal_ll(m, ' ', nice);
    seq_put_decimal_ll(m, ' ', num_threads);
    seq_put_decimal_ull(m, ' ', 0);
    seq_put_decimal_ull(m, ' ', start_time);
    seq_put_decimal_ull(m, ' ', vsize);
    seq_put_decimal_ull(m, ' ', mm ? get_mm_rss(mm) : 0);
    seq_put_decimal_ull(m, ' ', rsslim);
    seq_put_decimal_ull(m, ' ', mm ? (permitted ? mm->start_code : 1) : 0);
    seq_put_decimal_ull(m, ' ', mm ? (permitted ? mm->end_code : 1) : 0);
    seq_put_decimal_ull(m, ' ', (permitted && mm) ? mm->start_stack : 0);
    seq_put_decimal_ull(m, ' ', esp);
    seq_put_decimal_ull(m, ' ', eip);
    */
    rsize = sscanf(s, "%d %*s %*s %*s %*s %*s %*s %*s %*s %*s "
                      "%lu %lu %*s %*s %d %d %d %*s %lld %lu %lu "
                      "%*s %*s %*s %*s %lu %*s %*s %*s lu",
                      &(taskinfo->ppid), &(taskinfo->utime), &(taskinfo->stime),
                      &(taskinfo->priority), &(taskinfo->nice), &(taskinfo->num_threads),
                      &(taskinfo->starttime), &(taskinfo->vsize), &(taskinfo->rss),
                      &(taskinfo->rsslim), &(taskinfo->vsize), &(taskinfo->eip), &(taskinfo->wchan));

    free(stat_content);
    return rsize;
}
```

bionic/libc/bionic/grp_pwd.cpp
```c
static passwd* android_iinfo_to_passwd(passwd_state_t* state,
                                       const android_id_info* iinfo) {
  snprintf(state->name_buffer_, sizeof(state->name_buffer_), "%s", iinfo->name);
  snprintf(state->dir_buffer_, sizeof(state->dir_buffer_), "/");
  snprintf(state->sh_buffer_, sizeof(state->sh_buffer_), "/bin/sh");

  passwd* pw = &state->passwd_;
  pw->pw_uid   = iinfo->aid;
  pw->pw_gid   = iinfo->aid;
  return pw;
}

passwd* getpwuid_internal(uid_t uid, passwd_state_t* state) {
  if (auto* android_id_info = find_android_id_info(uid); android_id_info != nullptr) {
    return android_iinfo_to_passwd(state, android_id_info);
  }

  // Find an entry from the database file
  passwd* pw = oem_id_to_passwd(uid, state);
  if (pw != nullptr) {
    return pw;
  }
  return app_id_to_passwd(uid, state);
}

passwd* getpwuid(uid_t uid) {  // NOLINT: implementing bad function.
  passwd_state_t* state = get_passwd_tls_buffer();
  return getpwuid_internal(uid, state);
}

static void print_app_name_from_uid(const uid_t uid, char* buffer, const int bufferlen) {
  const uid_t appid = uid % AID_USER_OFFSET;
  const uid_t userid = uid / AID_USER_OFFSET;
  if (appid >= AID_ISOLATED_START) {
    snprintf(buffer, bufferlen, "u%u_i%u", userid, appid - AID_ISOLATED_START);
  } else if (appid < AID_APP_START) {
    if (auto* android_id_info = find_android_id_info(appid); android_id_info != nullptr) {
      snprintf(buffer, bufferlen, "u%u_%s", userid, android_id_info->name);
    }
  } else {
    snprintf(buffer, bufferlen, "u%u_a%u", userid, appid - AID_APP_START);
  }
}

// Translate a uid into the corresponding name.
// 0 to AID_APP_START-1                    -> "system", "radio", etc.
// AID_APP_START to AID_ISOLATED_START-1   -> u0_a1234
// AID_ISOLATED_START to AID_USER_OFFSET-1 -> u0_i1234
// AID_USER_OFFSET+                        -> u1_radio, u1_a1234, u2_i1234, etc.
// returns a passwd structure (sets errno to ENOENT on failure).
static passwd* app_id_to_passwd(uid_t uid, passwd_state_t* state) {
  if (uid < AID_APP_START || !is_valid_app_id(uid, false)) {
    errno = ENOENT;
    return nullptr;
  }

  print_app_name_from_uid(uid, state->name_buffer_, sizeof(state->name_buffer_));

  const uid_t appid = uid % AID_USER_OFFSET;
  if (appid < AID_APP_START) {
      snprintf(state->dir_buffer_, sizeof(state->dir_buffer_), "/");
  } else {
      snprintf(state->dir_buffer_, sizeof(state->dir_buffer_), "/data");
  }

  snprintf(state->sh_buffer_, sizeof(state->sh_buffer_), "/bin/sh");

  passwd* pw = &state->passwd_;
  pw->pw_uid   = uid;
  pw->pw_gid   = uid;
  return pw;
}
```
综上ux-axx的组成为u(userId)-a(appid - AID_APP_START):
app的uid/100000的结果为userid，填到ux的x处；app的uid % 100000的appid减去10000，填到axx的xx处。所以可以将ux-axx是由userId和appId的另一种表示为uid的形式。
