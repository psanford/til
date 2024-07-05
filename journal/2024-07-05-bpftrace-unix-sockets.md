bpftrace unix socket data.  Based on undump.bt:

```
#ifndef BPFTRACE_HAVE_BTF
#include <linux/skbuff.h>
#endif

kprobe:unix_stream_read_actor /comm=="rivercarro"/
{
  $skb = (struct sk_buff *)arg0;
  time("%H:%M:%S ");
  printf("recv %rx\n",  buf($skb->data, $skb->len));
}

tracepoint:syscalls:sys_enter_sendmsg /comm=="rivercarro"/
{
    $msghdr = (struct user_msghdr *)args.msg;
    $iov = (struct iovec *)$msghdr->msg_iov;

    if ($iov != 0 && $msghdr->msg_iovlen > 0) {
        $data = buf($iov->iov_base, $iov->iov_len > 16 ? $iov->iov_len : 16);
        time("%H:%M:%S ");
        printf("send %rx\n", $data);
    }
}

```
