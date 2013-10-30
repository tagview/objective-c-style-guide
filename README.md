# Tagview Objective-C Style Guide

Apple has already written a widely accepted coding guide for Objective-C. So, before reading this guide, please make sure you've read the [Apple's Cocoa Coding Guidelines](http://developer.apple.com/documentation/Cocoa/Conceptual/CodingGuidelines/index.html).

## Example

Below is an example that aggregates all the styles guidelines that we are gonna discuss on this document:

#### Particle.h
```objective-c
#import <Foundation/Foundation.h>
#include <Math.h>
#import "Spring.h"
#import "Vector.h"
#include "UtilityCFunctions.h"

typedef enum Solvers : NSUInteger {
    kEuler = 1,
    kVerlet = 2
} IntegrationMethod;

@interface Particle : NSObject <Springable>

@property (nonatomic, assign) CGFloat friction;
@property (nonatomic, strong) Vector *position;
@property (nonatomic, readonly) Vector *velocity;
@property (nonatomic, assign) IntegrationMethod integrationMethod;
@property (nonatomic, assign, getter = isFixed) BOOL fixed;

+ (Particle *)fixedParticleAtPosition:(Vector *)position;

- (void)initWithPosition:(Vector *)position;
- (void)initWithPosition:(Vector *)position velocity:(Vector *)velocity;

- (void)addForce:(Vector *)force;
- (void)integrate;

@end
```

#### Particle.m
```objective-c
@interface Particle ()

@property (nonatomic, strong) Vector *velocity;
@property (nonatomic, strong) Vector *acceleration;
@property (nonatomic, strong) Vector *previousPosition;

@end

@implementation Particle

#pragma mark - Initializers

// Designated initializer
- (id)initWithPosition:(Vector *)position velocity:(Vector *)velocity {
    if (self = [super init]) {
        _friction = 1;
        _position = position;
        _velocity = velocity;
        _acceleration = [[Vector alloc] init];
    }

    return self;
}

- (id)initWithPosition:(Vector *)position {
    Vector *velocity = [[Vector alloc] init];
    return [self initWithPosition:position velocity:velocity];
}


#pragma mark - Class methods

+ (Particle *)fixedParticleAtPosition:(Vector *)position {
    Particle *fixedParticle = [[Particle alloc] initWithPosition:position];
    fixedParticle.fixed = YES;

    return fixedParticle;
}


#pragma mark - Instance methods

- (void)addForce:(Vector *)force {
    [self.acceleration add:force];
}


#pragma mark - Getters and Setters

- (Vector *)previousPosition {
    if (!_previousPosition) {
        _previousPosition = self.position;
    }

    return _previousPosition;
}


#pragma mark - Integration

- (void)integrate {
    switch (self.integrationMethod) {
        case kEuler:
            [self eulerIntegration];
            break;

        case kVerlet:
            [self verletIntegration];
            break;

        default:
            [self eulerIntegration];
    }
}

- (void)eulerIntegration {
    [self.velocity add:self.acceleration];
    [self.velocity multiply:self.friction];
    [self.position add:self.velocity];
    [self.acceleration multiply:0];
}

- (void)verletIntegration {
    self.velocity = [Vector subtractVector:self.position withVector:self.previousPosition];
    self.previousPosition = [self.position clone];
    [self.velocity add:self.acceleration];
    [self.velocity multiply:self.friction];
    [self.position add:self.velocity];
    [self.acceleration multiply:0];
}

@end
```

## Guidelines

### Whitespace

Put one space before leading braces, `ifs`, `whiles`, `fors`, `switches`, and `@properties`.
```objective-c
// Bad
if(!_name){
    _name = @"Default";
}

// Good
if (!_name) {
    _name = @"Default";
}


// Bad
@property(nonatomic, strong)Vector *velocity;

// Good
@property (nonatomic, strong) Vector *velocity;
```

**Don't** put a spaces between parentheses.
```objective-c
// Bad
if ( !_name ) {
    _name = @"Default";
}

// Good
if (!_name) {
    _name = @"Default";
}
```

#### Method declaration and invocation

Put a space after the `-` or `+`.
```objective-c
// Bad
-(void)activate;

// Good
- (void)activate;
```

Put a space before the asterisks.
```objective-c
// Bad
- (void)setVelocity:(Vector*)velocity;

// Good
- (void)setVelocity:(Vector *)velocity;
```

**Don't** put a spaces on method arguments and on method declarations.
```objective-c
// Bad
User *user = [[User alloc] initWithFirstName: @"Rodrigo" lastName: @"Navarro"];

// Good
User *user = [[User alloc] initWithFirstName:@"Rodrigo" lastName:@"Navarro"];


// Bad
- (void)setVelocity: (Vector *)velocity;

// Good
- (void)setVelocity:(Vector *)velocity;
```

If you have too many parameters to fit on one line, giving each its own line is preferred. If multiple lines are used, align each using the colon before the parameter.
```objective-c
- (void)initWithPosition:(Vector *)position 
                velocity:(Vector *)velocity 
            acceleration:(Vector *)acceleration {
    ...
}

```

When the first keyword is shorter than the others, indent the later lines by at least four spaces, maintaining colon alignment:
```objective-c
- (void)position:(Vector *)position
    maximumVelocity:(Vector *)velocity
       acceleration:(Vector *)acceleration {
    ...
}
```

#### Container Literals

If the collection fits on one line, put a single space after the opening and before the closing brackets.
```objective-c
// Bad
NSArray *array = @[@"One", @"Two", @"Three"];
NSArray *array = @[@"One",@"Two",@"Three"];

// Good
NSArray *array = @[ @"One", @"Two", @"Three" ];


// Bad
NSDictionary *dict = @{@"firstName":@"Rodrigo", @"lastName":@"Navarro" };

// Good
NSDictionary *dict = @{ @"firstName": @"Rodrigo", @"lastName": @"Navarro" };
```

If the collection spans more than a single line, place the opening bracket on the same line as the declaration, indent the body by four spaces, and place the closing bracket on a new line that is indented to the same level as the opening bracket.
```objective-c
NSArray *array = @[
    @"One", 
    @"Two", 
    @"Three"
];
```

### Line Breaks

**Don't** put a line break after `ifs`, `whiles`, `fors`, and `switches`.
```objective-c
// Bad
if (!_name )
{
    _name = @"Default";
}

// Good
if (!_name) {
    _name = @"Default";
}
```

**Don't** put a line break before a method definition block.
```objective-c
// Bad
- (void)initWithPosition:(Vector *)position
{
    ...
}

// Good
- (void)initWithPosition:(Vector *)position {
    ...
}
```

Put line breaks between `@properties` and method definitions.
```objective-c
// Bad
@interface Particle : NSObject <Springable>
@property (nonatomic, assign, getter = isFixed) BOOL fixed;
+ (Particle *)fixedParticleAtPosition:(Vector *)position;

// Good
@interface Particle : NSObject <Springable>

@property (nonatomic, assign, getter = isFixed) BOOL fixed;

+ (Particle *)fixedParticleAtPosition:(Vector *)position;
```

### Properties

Always use properties instead of message passing when possible.
```objective-c
// Bad
if ([array count] > 10) {
    ...
}

// Good
if (array.count > 10) {
    ...
}

// Bad
[view setHidden:NO];

// Good
view.hidden = YES;
```

Don't use getters and setters during object initialization, instead, access the instance variables directly.
```objective-c
// Bad
- (id)initWithPosition:(Vector *)position velocity:(Vector *)velocity {
    if (self = [super init]) {
        self.position = position;
        self.velocity = velocity;
    }

    return self;
}

// Good
- (id)initWithPosition:(Vector *)position velocity:(Vector *)velocity {
    if (self = [super init]) {
        _position = position;
        _velocity = velocity;
    }

    return self;
}
```

### Import vs Include

Choose between `#import` and `#include` based on the language of the header that you are including.

- When including a header that uses Objective-C or Objective-C++, use `#import`
- When including a standard C or C++ header, use `#include`

```objective-c
#import <Cocoa/Cocoa.h>
#include <CoreFoundation/CoreFoundation.h>
#import "GTMFoo.h"
#include "base/basictypes.h"
```

For more information on this topic, see [this Stackoverflow answer on the subject](http://stackoverflow.com/a/4219472).
