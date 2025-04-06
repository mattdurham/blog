# Objective C

Date: 2011-09-30 12:00:00

Update 2011-06-16 I noticed I have a memory leak with the callback, updated to fix that issue

I have been coding for a few months now and in the past few weeks found blocks, I found them extremely useful given the asynchronous nature of my code. Attached is a very simple Xcode project that shows how to use blocks. Some quick observations before we dive in.

    Always use typedefs when defining the block definitions
    When using blocks for callbacks, ensure you consider the tradeoffs between using a delegate instead ** A delegate allows you to get information from the ‘parent’, you could with a block but its a bit clunky ** A delegate is better when you have lots of communication going back and forth, when I use a callback I generally supply a done and an error callback.
    If storing a callback you need to COPY from the original callback and then release your local copy of the callback, this is because generally callbacks are stored on the stack, when you copy they go on the heap like most other objective-c objects
    I have only been using blocks for a few weeks so take anything I say with a grain of salt and be sure to read the original Apple documentation and other resources on the web

I fear the actual code is somewhat unimpressive, I typedef the callback and it takes one parameter, an int telling it how long it waited.

```
```
#import <Foundation/Foundation.h>
typedef void (^KORCallback)(int wait);
@interface KORBlocks : NSObject 
{
    KORCallback callback_;
}
-(id) initWithCallback:(KORCallback) callback;
-(void) startExecutingAsyncAction;
@end

Here is the implementation, nothing fancy, notice the copy on the initWithCallback command, and then we just use GCD (dispatch_async) commands to execute the commands, using GCD and blocks gives programs a nice way to handle multithreaded programs, this keeps your UI responsive and makes the users happy.

@implementation KORBlocks

-(id) initWithCallback:(KORCallback) callback
{
    self = [super init];
    //This is important that you copy the callback, when it comes down to you it is most likely
    //  stack alloced and calling copy moves it to the heap, also note you need to release it in the dealloc
    callback_ = [callback copy];
    return self;
}

-(void) dealloc
{

    [callback_ release];
    [super dealloc];
}

-(void) startExecutingAsyncAction
{
    //Start sleeping on another Queue when that is done call the callback on the main gui thread
    dispatch_async(dispatch_get_global_queue(0,0), ^(void)
       {
        int r = arc4random() % 10;
        //Sleep for r seconds then call the callback on the main thread
        [NSThread sleepForTimeInterval:r]; 
        dispatch_async(dispatch_get_main_queue(), ^(void)
        {
               callback_(r);
               [callback_ release]
               callback_ = nil;
        });
    });
}

@end

```
```

Images of the project in action
