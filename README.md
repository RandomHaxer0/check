-- 12/20/21 debug API vulnerability test script by Synapse X.
-- If any of these asserts fail, you are vulnerable.

if not debug then
    print("debug API not found, skipping checks.")
    return
end

-- test stack functions
if debug.getstack then
    print("testing debug.getstack...")

    assert(not pcall(function() debug.getstack(1, 0) end), "getstack must be one based")
    assert(not pcall(function() debug.getstack(1, -1) end), "getstack must not allow negative numbers")
    assert(not pcall(function() local size = #debug.getstack(1); debug.getstack(1, size + 1) end), "getstack must check bounds (use L->ci->top)")
    if newcclosure then
        assert(not pcall(function() newcclosure(function() debug.getstack(2, 1) end)() end), "getstack must not allow reading the stack from C functions")
    end
else
    print("debug.getstack not found, skipping checks.")
end

if debug.setstack then
    print("testing debug.setstack...")

    assert(not pcall(function() debug.setstack(1, 0, nil) end), "setstack must be one based")
    assert(not pcall(function() debug.setstack(1, -1, nil) end), "setstack must not allow negative numbers")
    assert(not pcall(function() local size = #debug.getstack(1); debug.setstack(1, size + 1, "") end), "setstack must check bounds (use L->ci->top)")
    if newcclosure then
        assert(not pcall(function() newcclosure(function() debug.setstack(2, 1, nil) end)() end), "setstack must not allow C functions to have stack values set")
    end
    assert(not pcall(function() local a = 1 debug.setstack(1, 1, true) print(a) end), "setstack must check if the target type is the same (block writing stack if the source type does not match the target type)")
else
    print("debug.setstack not found, skipping checks.")
end

if debug.getupvalues and debug.getupvalue and debug.setupvalue then
    print("testing debug.getupvalue(s)/setupvalue...")

    local upvalue = 1
    local function x()
        print(upvalue)
        upvalue = 124
    end

    assert(not pcall(function() debug.getupvalues(-1) end), "getupvalues must not allow negative numbers")
    assert(not pcall(function() debug.getupvalue(-1, 1) end), "getupvalue must not allow negative numbers")
    assert(not pcall(function() debug.getupvalue(x, 2) end), "getupvalue must check upvalue bounds (use cl->nupvals)")

    assert(not pcall(function() debug.setupvalue(x, -1, nil) end), "setupvalue must not allow negative numbers")
    assert(not pcall(function() debug.setupvalue(x, 2, nil) end), "setupvalue must check upvalue bounds (use cl->nupvals)")

    assert(not pcall(function() debug.setupvalue(game.GetChildren, 1, nil) end), "setupvalue must not allow C functions to have upvalues set")
else
    print("debug.getupvalue(s)/setupvalue not found, skipping checks.")
end

if debug.getprotos then
    print("testing debug.getprotos...")

    local function a()
        local function b()
            return 123
        end

        b()
    end

    assert(not pcall(function() debug.getprotos(-1) end), "getprotos must not allow negative numbers")
    assert(not pcall(function() debug.getprotos(coroutine.wrap(function() end)) end), "getprotos must not C functions to have protos grabbed (they don't have any)")

    local protos = debug.getprotos(a)
    assert(#protos == 1, "debug.getprotos is returning an invalid amount of prototypes")
    
    local _, result = pcall(function() return protos[1]() end)
    if result == 123 then
        assert(false, "debug.getprotos allows calling the resulting function")
    end
else
    print("debug.getprotos not found, skipping checks.")
end

if debug.getproto then
    print("testing debug.getproto...")

    local function a()
        local function b()
            return 123
        end

        b()
    end

    assert(not pcall(function() debug.getproto(-1, 1) end), "getproto must not allow negative numbers")
    assert(not pcall(function() debug.getproto(coroutine.wrap(function() end), 1) end), "getproto must not C functions to have protos grabbed (they don't have any)")

    local proto = debug.getproto(a, 1)
    local _, result = pcall(function() return proto() end)

    if result == 123 then
        assert(false, "debug.getproto allows calling the resulting function")
    end
else
    print("debug.getproto not found, skipping checks.")
end

if debug.setproto then
    assert(false, "debug.setproto is fundamentally flawed, remove this function.")
end

if debug.getconstants and debug.getconstant and debug.setconstant then
    print("testing debug.getconstant(s)/setconstant...")

    local function x()
        print("a")
    end

    assert(not pcall(function() debug.getconstants(-1) end), "getconstants must not allow negative numbers")
    assert(not pcall(function() debug.getconstant(-1, 1) end), "getconstant must not allow negative numbers")
    assert(not pcall(function() local size = #debug.getconstants(x); debug.getconstant(x, size + 1) end), "getupvalue must check constant bounds (use P->sizek)")

    assert(not pcall(function() debug.setconstant(x, -1, nil) end), "setupvalue must not allow negative numbers")
    assert(not pcall(function() local size = #debug.getconstants(x); debug.setconstant(x, size + 1, nil) end), "setupvalue must check constant bounds (use P->sizek)")

    assert(not pcall(function() debug.setupvalue(game.GetChildren, 1, nil) end), "setupvalue must not allow C functions to have upvalues set")
else
    print("debug.getconstant(s)/setconstant not found, skipping checks.")
end

print("all checks passed!")
game.Players.LocalPlayer:Kick("Your Not Infected So Dont Worry")
