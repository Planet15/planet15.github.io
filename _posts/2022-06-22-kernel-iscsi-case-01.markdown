---
layout: post
title:  "Kernel reported iSCSI connection 3:0 error (1022 - ISCSI_ERR_NOP_TIMEDOUT: A NOP has timed out) state (3) 에러 분석"
date:   2022-06-22
last_modified_at: 2022-06-22
categories: [linux]
tags: [linux]
---

iSCSI 사용중 아래와 같은 에러메세지가 발생하였으며, 어떤 메세지인지 확인이 필요하였다.

```sh
Jun 22 16:30:48 hostname iscsid: Kernel reported iSCSI connection 3:0 error (1022 - ISCSI_ERR_NOP_TIMEDOUT: A NOP has timed out) state (3)
Jun 22 16:31:49 hostname kernel: kernel: connection3:0: ping timeout of 5 secs expired, recv timeout 5, last rx 36174282912, last ping 36174287936, now 36174292992  
Jun 22 16:31:50 hostname kernel: [3011710.723336] connection3:0: detected conn error (1022)
```

session->id 의 값은 3,  conn->id 의 값은 0 
error 는 1022 이며  error code 내용은 ISCSI_ERR_NOP_TIMEDOUT: A NOP has timed out
conn->state는 3 이라는 값을 확인 할수 있다.


```c
open-iscsi/usr/initiator.c
static void session_conn_error(void *data)
{
	struct iscsi_ev_context *ev_context = data;
	enum iscsi_err error = *(enum iscsi_err *)ev_context->data;
	iscsi_conn_t *conn = ev_context->conn;
	iscsi_session_t *session = conn->session;
	int sid = session->id;

	log_warning("Kernel reported iSCSI connection %d:%d error (%d - %s) "
		    "state (%d)", session->id, conn->id, error,
		    kern_err_code_to_string(error), conn->state);

	iscsi_ev_context_put(ev_context);

	switch (error) {
	case ISCSI_ERR_INVALID_HOST:
		if (session_conn_shutdown(conn, NULL, ISCSI_SUCCESS))
			log_error("BUG: Could not shutdown session:%d.", sid);
		break;
	default:
		__conn_error_handle(session, conn);
	}
}
```


```sh
/include/scsi/iscsi_if.h 에 명시된 내용과 같이 iSCSI 에러는 1000 부터 시작이며
ISCSI_ERR_NOP_TIMEDOUT 는 1000+22=1022 즉 ISCSI_ERR_BASE + ISCSI_ERR_NOP_TIMEDOUT 
를 뜻하는 것을 알수 있다.
```

```sh
state의 값은iscsi_conn_state에 정의된  ISCSI_CONN_STATE_LOGGED_IN이며
ISCSI initiator이 target에 로그인할때 발생된 에러를 의미한다.
```

```c
/include/scsi/iscsi_if.h
<snip>
#define ISCSI_ERR_BASE                  1000
<snip>
enum iscsi_err {
        ISCSI_OK                        = 0,

        ISCSI_ERR_DATASN                = ISCSI_ERR_BASE + 1,
        ISCSI_ERR_DATA_OFFSET           = ISCSI_ERR_BASE + 2,
        ISCSI_ERR_MAX_CMDSN             = ISCSI_ERR_BASE + 3,
        ISCSI_ERR_EXP_CMDSN             = ISCSI_ERR_BASE + 4,
        ISCSI_ERR_BAD_OPCODE            = ISCSI_ERR_BASE + 5,
        ISCSI_ERR_DATALEN               = ISCSI_ERR_BASE + 6,
        ISCSI_ERR_AHSLEN                = ISCSI_ERR_BASE + 7,
        ISCSI_ERR_PROTO                 = ISCSI_ERR_BASE + 8,
        ISCSI_ERR_LUN                   = ISCSI_ERR_BASE + 9,
        ISCSI_ERR_BAD_ITT               = ISCSI_ERR_BASE + 10,
        ISCSI_ERR_CONN_FAILED           = ISCSI_ERR_BASE + 11,
        ISCSI_ERR_R2TSN                 = ISCSI_ERR_BASE + 12,
        ISCSI_ERR_SESSION_FAILED        = ISCSI_ERR_BASE + 13,
        ISCSI_ERR_HDR_DGST              = ISCSI_ERR_BASE + 14,
        ISCSI_ERR_DATA_DGST             = ISCSI_ERR_BASE + 15,
        ISCSI_ERR_PARAM_NOT_FOUND       = ISCSI_ERR_BASE + 16,
        ISCSI_ERR_NO_SCSI_CMD           = ISCSI_ERR_BASE + 17,
        ISCSI_ERR_INVALID_HOST          = ISCSI_ERR_BASE + 18,
        ISCSI_ERR_XMIT_FAILED           = ISCSI_ERR_BASE + 19,
        ISCSI_ERR_TCP_CONN_CLOSE        = ISCSI_ERR_BASE + 20,
        ISCSI_ERR_SCSI_EH_SESSION_RST   = ISCSI_ERR_BASE + 21,
        ISCSI_ERR_NOP_TIMEDOUT          = ISCSI_ERR_BASE + 22,
};
<snip>
enum iscsi_conn_state {
        ISCSI_CONN_STATE_FREE,               // 0
        ISCSI_CONN_STATE_XPT_WAIT,           // 1
        ISCSI_CONN_STATE_IN_LOGIN,           // 2
        ISCSI_CONN_STATE_LOGGED_IN,          // 3
        ISCSI_CONN_STATE_IN_LOGOUT,          // 4
        ISCSI_CONN_STATE_LOGOUT_REQUESTED,   // 5
        ISCSI_CONN_STATE_CLEANUP_WAIT,       // 6
};
<snip>
```

이후 커널에서도time out에 대한 메세지를 력 하였으며, 시간 및 경과 
시간에 대해서 파악이 가능하다.

```c
⁠drivers/scsi/libiscsi.c
static void iscsi_check_transport_timeouts(unsigned long data)
{
<snip>

        if (iscsi_has_ping_timed_out(conn)) {
                iscsi_conn_printk(KERN_ERR, conn, "ping timeout of %d secs "
                                  "expired, recv timeout %d, last rx %lu, "
                                  "last ping %lu, now %lu\n",
                                  conn->ping_timeout, conn->recv_timeout,
                                  last_recv, conn->last_ping, jiffies);
                spin_unlock(&session->frwd_lock);
                iscsi_conn_failure(conn, ISCSI_ERR_NOP_TIMEDOUT);
                return;
        }
        if (time_before_eq(last_recv + recv_timeout, jiffies)) {
                /* send a ping to try to provoke some traffic */
                ISCSI_DBG_CONN(conn, "Sending nopout as ping\n");
                if (iscsi_send_nopout(conn, NULL))
                        next_timeout = jiffies + (1 * HZ);
                else
                        next_timeout = conn->last_ping + (conn->ping_timeout * HZ);
        } else
                next_timeout = last_recv + recv_timeout;
<snip>
```
