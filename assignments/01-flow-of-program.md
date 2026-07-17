[Video Link](https://youtu.be/lhELGQAV4gg)

## Create flowchart and pseudocode for the following:

1. Input a year and find whether it is a leap year or not.
```rust
use std::io;

fn main(){
    let number: i32 = loop {
        
    println!("please enter a number : ");
    let mut input =String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("please enter number");
        
        match input.trim().parse(){
            Ok(num)=> break num,
            Err(_)=>{
                println!("please enter a valid number");
                continue;
            }
        }
    };
        
    let mut flag = false;
    if (number % 4 ==0  && number % 100 != 0 )|| number % 400 ==0 {
        flag = true;
    }
    println!("this is leap year? {}", flag);
}
```

2. Take two numbers and print the sum of both.
3. Take a number as input and print the multiplication table for it.
4. Take 2 numbers as inputs and find their HCF and LCM.
5. Keep taking numbers as inputs till the user enters ‘x’, after that print sum of all.
