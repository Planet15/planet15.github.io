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
