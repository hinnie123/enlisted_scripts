# enlisted_scripts
This repo contains all the [daScript](https://github.com/GaijinEntertainment/daScript) and [quirrel](https://github.com/GaijinEntertainment/quirrel) scripts used in the game Enlisted.

## Update
I've removed the scripts from the repository as they were in conflict with their copyright. The method to dump the scripts however, still works. Refer to the code below to dump the scripts yourself.

## Info
The game Enlisted uses two scripting engines, one named daScript, and the other is their own interpretation of squirrel, quirrel.
This repository contains all the scripts the game uses from both scripting engines, dumped myself.

## Dumping daScript
You can dump the daScript scripts by hooking ["FileAccess::getFileInfo"](https://github.com/GaijinEntertainment/daScript/blob/17fd6256e6c303ab78cbad87c3093357e0af35c7/src/simulate/debug_info.cpp#L507). (E8 ? ? ? ? 48 85 C0 74 2E 48 89 C6)
```
struct file_info_t 
{
	char pad[0x20];
	char* source;
	uint32_t source_length;
};

typedef file_info_t*(__fastcall* get_file_info_t)(void*, string);
get_file_info_t o_get_file_info;

file_info_t* __fastcall hk_get_file_info(void* thisptr, string file_name)
{
	file_info_t* info = o_get_file_info(thisptr, file_name);

	std::string name = file_name;
	size_t pos = name.find('./');
	if (pos != std::string::npos)
		name = replace_all(name.substr(pos + 1), "/", "\\");

	if (MakeSureDirectoryPathExists(std::string("C:\\path\\to\\dumps\\dascript\\" + name).c_str()))
	{
		std::ofstream file("C:\\path\\to\\dumps\\dascript\\" + name, std::ofstream::out);
		file << std::string(info->source, info->source + info->source_length);
		file.close();
	}

	return info;
}
```

## Dumping quirrel
You can dump the quirrel scripts by hooking ["sq_compile"](https://github.com/GaijinEntertainment/quirrel/blob/master/squirrel/sqapi.cpp#L144). (E8 ? ? ? ? 48 89 F1 48 85 C0 78 09)
```
struct buf_state_t
{
	const char* buf;
	uintptr_t ptr;
	uintptr_t size;
};

typedef uintptr_t(__fastcall sq_compile_t)(void*, void*, buf_state_t*, char*, bool, uintptr_t);
sq_compile_t* o_sq_compile;

uintptr_t __fastcall hk_sq_compile(void* v, void* read, buf_state_t* p, char* source_name, bool raise_error, uintptr_t unk)
{
	std::string name = replace_all(std::string(source_name), "/", "\\");
	if (MakeSureDirectoryPathExists(std::string("C:\\path\\to\\dumps\\quirrel\\" + name).c_str()))
	{
		std::ofstream file("C:\\path\\to\\dumps\\quirrel\\" + name, std::ofstream::out);
		file << std::string(p->buf, p->buf + p->size);
		file.close();
	}

	return o_sq_compile(v, read, p, source_name, raise_error, unk);
}
```
