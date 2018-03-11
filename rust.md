Casting data to a struct, example:

    #[repr(C, packed)]
    struct my_struct {
        foo: u16,
        bar: u8,
    }

    fn main() {
        let v: Vec<u8> = vec![1, 2, 3];
        let s: MyStruct = unsafe { std::ptr::read(v.as_ptr() as *const _) };
        println!("here is the struct: {:?}", s);
    }
