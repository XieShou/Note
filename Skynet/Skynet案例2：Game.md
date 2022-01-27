# Skynet案例2：Game

以下开始讲`local gate = skynet.newservice("gated", loginserver)`开启游戏服务器的流程。

## msgserver

```lua
function server.start(conf)
	local expired_number = conf.expired_number or 128

	local handler = {}

	local CMD = {
		login = assert(conf.login_handler),
		logout = assert(conf.logout_handler),
		kick = assert(conf.kick_handler),
	}

	function handler.command(cmd, source, ...) ... end
	function handler.open(source, gateconf) ... end
	function handler.connect(fd, addr) ... end
	function handler.disconnect(fd) ... end
	handler.error = handler.disconnect

	-- atomic , no yield
	local function do_auth(fd, message, addr) ... end
	local function auth(fd, addr, msg, sz) ... end
	local request_handler = assert(conf.request_handler)

	-- u.response is a struct { return_fd , response, version, index }
	local function retire_response(u) ... end
	local function do_request(fd, message) ... end
	local function request(fd, msg, sz) ... end
	function handler.message(fd, msg, sz) ... end
	return gateserver.start(handler)
end
```

先不用看具体逻辑，可以这个函数在最后又调用了`gateserver.start`函数

```lua
function gateserver.start(handler)
	assert(handler.message)
	assert(handler.connect)

	function CMD.open(source, conf) ... end

	function CMD.close() ... end

	local MSG = {}

	local function dispatch_msg(fd, msg, sz) ... end

	MSG.data = dispatch_msg

	local function dispatch_queue() ... end

	MSG.more = dispatch_queue

	function MSG.open(fd, msg) ... end

	function MSG.close(fd) ... end

	function MSG.error(fd, msg) ... end

	function MSG.warning(fd, size) ... end

	skynet.register_protocol {
		name = "socket",
		id = skynet.PTYPE_SOCKET,	-- PTYPE_SOCKET = 6
		unpack = function ( msg, sz )
			return netpack.filter( queue, msg, sz)
		end,
		dispatch = function (_, _, q, type, ...)
			queue = q
			if type then
				MSG[type](...)
			end
		end
	}

	local function init()
		skynet.dispatch("lua", function (_, address, cmd, ...)
			local f = CMD[cmd]
			if f then
				skynet.ret(skynet.pack(f(address, ...)))
			else
				skynet.ret(skynet.pack(handler.command(cmd, address, ...)))
			end
		end)
	end

	if handler.embed then
		init()
	else
		skynet.start(init)
	end
end
```















基于上一篇的案例——Login模块中，在登陆的最后通知了gameserver玩家登陆。

```lua
function server.login_handler(server, uid, secret)
    print(string.format("%s@%s is login, secret is %s", uid, server, crypt.hexencode(secret)))
    local gameserver = assert(server_list[server], "Unknown server")
    -- only one can login, because disallow multilogin
    local last = user_online[uid]
    if last then
        skynet.call(last.address, "lua", "kick", uid, last.subid)
    end
    if user_online[uid] then
        error(string.format("user %s is already online", uid))
    end
    --在这里开始通知
    local subid = tostring(skynet.call(gameserver, "lua", "login", uid, secret))
    user_online[uid] = { address = gameserver, subid = subid , server = server}
    return subid
end
```
