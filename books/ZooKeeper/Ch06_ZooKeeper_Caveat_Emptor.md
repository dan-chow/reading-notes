## Chapter 06: ZooKeeper Caveat Emptor

- Normally you would expect access control to be described in an administration section. However, in ZooKeeper the developer, rather than the administrator, is usually the one who manages access control. This is because access rights must be set every time a znode is created; it does not inherit the access permissions from its parent. Access checks are also done on a per-znode basis. If a client has access to a znode, it can access it even if that client cannot access the parent of the znode.

	ZooKeeper controls access using access control lists (ACLs). An ACL contains entries of the form scheme:auth-info, where scheme corresponds to a set of built-in authentication schemes and auth-info encodes the authentication information in some manner specific to the scheme.

	To add authentication information to a ZooKeeper handle, issue the addAuthInfo call in the format:
  ```java
  void addAuthInfo(String scheme, byte auth[])
  ```

- ZooKeeper offers four built-in schemes to handle ACLs. One of them we have been using implicitly through the OPEN_ACL_UNSAFE constant. That ACL uses the world scheme that just lists anyone as the auth-info. anyone is the only auth-info that can be used with the world scheme.

	Another special built-in scheme used by administrators is the super scheme. This scheme is never included in any ACL, but it can be used to authenticate to ZooKeeper. A client that authenticates with super will not be restricted by ACLs of any of the znodes.

	digest is a built-in scheme whose auth-info has the form userid:passwd_digest when setting the ACL and userid:password when calling addAuthInfo. The passwd_digest is a cryptographic digest of the user’s password.

	The ip scheme takes the network address and mask. Because it uses the address of the client to do the ACL check, clients do not need to call addAuthInfo with the ip scheme to access a znode using this ACL.

- SASL stands for Simple Authentication and Security Layer. It is a framework that abstracts the underlying system of authentication so that applications that use SASL can use any of the various protocols supported by SASL. With respect to ZooKeeper, SASL usually uses Kerberos, which is an authentication protocol that provides the missing features that we mentioned earlier. SASL uses sasl for its scheme name, and the id is the Kerberos ID of the client.

- If clients of an application communicate only by reading and writing to ZooKeeper, the application shouldn’t worry about sync. sync exists because communication outside ZooKeeper may lead to a problem often referred to as a hidden channel.

	sync is an asynchronous call that a client uses before a read operation. Say that a client wants to read the znode that it has heard through a direct channel has changed. The client calls sync, followed by getData:
  ```java
  ...
  zk.sync(path, voidCb, ctx);
  zk.getData(path, watcher, dataCb, ctx);
  ...
  ```