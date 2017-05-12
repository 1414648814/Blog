


```cpp

// IPv4套接字地址结构
struct sockaddr_in {
	__uint8_t	sin_len;
	sa_family_t	sin_family;
	in_port_t	sin_port;
	struct	in_addr sin_addr;
	char		sin_zero[8];
};

// 通用套接字地址结构
struct sockaddr {
	__uint8_t	sa_len;		/* total length */
	sa_family_t	sa_family;	/* [XSI] address family */
	char		sa_data[14];	/* [XSI] addr value (actually larger) */
};

```

---

IPv6

```cpp
struct sockaddr_in6 {
	__uint8_t	sin6_len;	/* length of this struct(sa_family_t) */
	sa_family_t	sin6_family;	/* AF_INET6 (sa_family_t) */
	in_port_t	sin6_port;	/* Transport layer port # (in_port_t) */
	__uint32_t	sin6_flowinfo;	/* IP6 flow information */
	struct in6_addr	sin6_addr;	/* IP6 address */
	__uint32_t	sin6_scope_id;	/* scope zone index */
};

struct sockaddr_storage {
	__uint8_t	ss_len;		/* address length */
	sa_family_t	ss_family;	/* [XSI] address family */
	char			__ss_pad1[_SS_PAD1SIZE];
	__int64_t	__ss_align;	/* force structure storage alignment */
	char			__ss_pad2[_SS_PAD2SIZE];
};

```

sockaddr_storage 对比 sockaddr：

1. 满足任何苛刻的对齐需要
2. 足够大，容纳任何套接字地址结构

