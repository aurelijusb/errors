
# errors
    import "github.com/juju/errors"

[![GoDoc](https://godoc.org/github.com/juju/errors?status.svg)](https://godoc.org/github.com/juju/errors)

The juju/errors provides an easy way to annotate errors without losing the
orginal error context.

The exported `New` and `Errorf` functions are designed to replace the
`errors.New` and `fmt.Errorf` functions respectively. The same underlying
error is there, but the package also records the location at which the error
was created.

A primary use case for this library is to add extra context any time an
error is returned from a function.


	    if err := SomeFunc(); err != nil {
		    return err
		}

This instead becomes:


	    if err := SomeFunc(); err != nil {
		    return errors.Trace(err)
		}

which just records the file and line number of the Trace call, or


	    if err := SomeFunc(); err != nil {
		    return errors.Annotate(err, "more context")
		}

which also adds an annotation to the error.

When you want to check to see if an error is of a particular type, a helper
function is normally exported by the package that returned the error, like the
`os` package does.  The underlying cause of the error is available using the
`Cause` function.


	os.IsNotExist(errors.Cause(err))

The result of the `Error()` call on an annotated error is the annotations joined
with colons, then the result of the `Error()` method for the underlying error
that was the cause.


	err := errors.Errorf("original")
	err = errors.Annotatef(err, "context")
	err = errors.Annotatef(err, "more context")
	err.Error() -> "more context: context: original"

Obviously recording the file, line and functions is not very useful if you
cannot get them back out again.


	errors.ErrorStack(err)

will return something like:


	first error
	github.com/juju/errors/annotation_test.go:193:
	github.com/juju/errors/annotation_test.go:194: annotation
	github.com/juju/errors/annotation_test.go:195:
	github.com/juju/errors/annotation_test.go:196: more context
	github.com/juju/errors/annotation_test.go:197:

The first error was generated by an external system, so there was no location
associated. The second, fourth, and last lines were generated with Trace calls,
and the other two through Annotate.

Sometimes when responding to an error you want to return a more specific error
for the situation.


	    if err := FindField(field); err != nil {
		    return errors.Wrap(err, errors.NotFoundf(field))
		}

This returns an error where the complete error stack is still available, and
`errors.Cause()` will return the `NotFound` error.






## func AlreadyExistsf
``` go
func AlreadyExistsf(format string, args ...interface{}) error
```
AlreadyExistsf returns an error which satisfies IsAlreadyExists().


## func Annotate
``` go
func Annotate(other error, message string) error
```
Annotate is used to add extra context to an existing error. The location of
the Annotate call is recorded with the annotations. The file, line and
function are also recorded.

For example:


	if err := SomeFunc(); err != nil {
	    return errors.Annotate(err, "failed to frombulate")
	}


## func Annotatef
``` go
func Annotatef(other error, format string, args ...interface{}) error
```
Annotatef is used to add extra context to an existing error. The location of
the Annotate call is recorded with the annotations. The file, line and
function are also recorded.

For example:


	if err := SomeFunc(); err != nil {
	    return errors.Annotatef(err, "failed to frombulate the %s", arg)
	}


## func BadRequestf
``` go
func BadRequestf(format string, args ...interface{}) error
```
BadRequestf returns an error which satisfies IsBadRequest().


## func Cause
``` go
func Cause(err error) error
```
Cause returns the cause of the given error.  This will be either the
original error, or the result of a Wrap or Mask call.

Cause is the usual way to diagnose errors that may have been wrapped by
the other errors functions.


## func CauseEquals
``` go
func CauseEquals(err1, err2 error) bool
```
CauseEquals tests, if this or any cause of this error is equal to another error, or it's cause

For example:


	ErrNotFound := errors.New("Not found")
	inSome := errors.Trace(ErrNotFound)
	veryDeep := errors.Trace(inSome)
	returned := errors.Trace(veryDeep)
	if errors.CauseEquals(ErrNotFound, returned) {
		fmt.Println("Can return soft error 404")
	} else {
		fmt.Println("Serious error, returning with code 500")
	}


## func DeferredAnnotatef
``` go
func DeferredAnnotatef(err *error, format string, args ...interface{})
```
DeferredAnnotatef annotates the given error (when it is not nil) with the given
format string and arguments (like fmt.Sprintf). If *err is nil, DeferredAnnotatef
does nothing. This method is used in a defer statement in order to annotate any
resulting error with the same message.

For example:


	defer DeferredAnnotatef(&err, "failed to frombulate the %s", arg)


## func Details
``` go
func Details(err error) string
```
Details returns information about the stack of errors wrapped by err, in
the format:


	[{filename:99: error one} {otherfile:55: cause of error one}]

This is a terse alternative to ErrorStack as it returns a single line.


## func ErrorStack
``` go
func ErrorStack(err error) string
```
ErrorStack returns a string representation of the annotated error. If the
error passed as the parameter is not an annotated error, the result is
simply the result of the Error() method on that error.

If the error is an annotated error, a multi-line string is returned where
each line represents one entry in the annotation stack. The full filename
from the call stack is used in the output.


	first error
	github.com/juju/errors/annotation_test.go:193:
	github.com/juju/errors/annotation_test.go:194: annotation
	github.com/juju/errors/annotation_test.go:195:
	github.com/juju/errors/annotation_test.go:196: more context
	github.com/juju/errors/annotation_test.go:197:


## func Errorf
``` go
func Errorf(format string, args ...interface{}) error
```
Errorf creates a new annotated error and records the location that the
error is created.  This should be a drop in replacement for fmt.Errorf.

For example:


	return errors.Errorf("validation failed: %s", message)


## func IsAlreadyExists
``` go
func IsAlreadyExists(err error) bool
```
IsAlreadyExists reports whether the error was created with
AlreadyExistsf() or NewAlreadyExists().


## func IsBadRequest
``` go
func IsBadRequest(err error) bool
```
IsBadRequest reports whether err was created with BadRequestf() or
NewBadRequest().


## func IsMethodNotAllowed
``` go
func IsMethodNotAllowed(err error) bool
```
IsMethodNotAllowed reports whether err was created with MethodNotAllowedf() or
NewMethodNotAllowed().


## func IsNotAssigned
``` go
func IsNotAssigned(err error) bool
```
IsNotAssigned reports whether err was created with NotAssignedf() or
NewNotAssigned().


## func IsNotFound
``` go
func IsNotFound(err error) bool
```
IsNotFound reports whether err was created with NotFoundf() or
NewNotFound().


## func IsNotImplemented
``` go
func IsNotImplemented(err error) bool
```
IsNotImplemented reports whether err was created with
NotImplementedf() or NewNotImplemented().


## func IsNotProvisioned
``` go
func IsNotProvisioned(err error) bool
```
IsNotProvisioned reports whether err was created with NotProvisionedf() or
NewNotProvisioned().


## func IsNotSupported
``` go
func IsNotSupported(err error) bool
```
IsNotSupported reports whether the error was created with
NotSupportedf() or NewNotSupported().


## func IsNotValid
``` go
func IsNotValid(err error) bool
```
IsNotValid reports whether the error was created with NotValidf() or
NewNotValid().


## func IsUnauthorized
``` go
func IsUnauthorized(err error) bool
```
IsUnauthorized reports whether err was created with Unauthorizedf() or
NewUnauthorized().


## func IsUserNotFound
``` go
func IsUserNotFound(err error) bool
```
IsUserNotFound reports whether err was created with UserNotFoundf() or
NewUserNotFound().


## func Mask
``` go
func Mask(other error) error
```
Mask hides the underlying error type, and records the location of the masking.


## func Maskf
``` go
func Maskf(other error, format string, args ...interface{}) error
```
Mask masks the given error with the given format string and arguments (like
fmt.Sprintf), returning a new error that maintains the error stack, but
hides the underlying error type.  The error string still contains the full
annotations. If you want to hide the annotations, call Wrap.


## func MethodNotAllowedf
``` go
func MethodNotAllowedf(format string, args ...interface{}) error
```
MethodNotAllowedf returns an error which satisfies IsMethodNotAllowed().


## func New
``` go
func New(message string) error
```
New is a drop in replacement for the standard libary errors module that records
the location that the error is created.

For example:


	return errors.New("validation failed")


## func NewAlreadyExists
``` go
func NewAlreadyExists(err error, msg string) error
```
NewAlreadyExists returns an error which wraps err and satisfies
IsAlreadyExists().


## func NewBadRequest
``` go
func NewBadRequest(err error, msg string) error
```
NewBadRequest returns an error which wraps err that satisfies
IsBadRequest().


## func NewMethodNotAllowed
``` go
func NewMethodNotAllowed(err error, msg string) error
```
NewMethodNotAllowed returns an error which wraps err that satisfies
IsMethodNotAllowed().


## func NewNotAssigned
``` go
func NewNotAssigned(err error, msg string) error
```
NewNotAssigned returns an error which wraps err that satisfies
IsNotAssigned().


## func NewNotFound
``` go
func NewNotFound(err error, msg string) error
```
NewNotFound returns an error which wraps err that satisfies
IsNotFound().


## func NewNotImplemented
``` go
func NewNotImplemented(err error, msg string) error
```
NewNotImplemented returns an error which wraps err and satisfies
IsNotImplemented().


## func NewNotProvisioned
``` go
func NewNotProvisioned(err error, msg string) error
```
NewNotProvisioned returns an error which wraps err that satisfies
IsNotProvisioned().


## func NewNotSupported
``` go
func NewNotSupported(err error, msg string) error
```
NewNotSupported returns an error which wraps err and satisfies
IsNotSupported().


## func NewNotValid
``` go
func NewNotValid(err error, msg string) error
```
NewNotValid returns an error which wraps err and satisfies IsNotValid().


## func NewUnauthorized
``` go
func NewUnauthorized(err error, msg string) error
```
NewUnauthorized returns an error which wraps err and satisfies
IsUnauthorized().


## func NewUserNotFound
``` go
func NewUserNotFound(err error, msg string) error
```
NewUserNotFound returns an error which wraps err and satisfies
IsUserNotFound().


## func NotAssignedf
``` go
func NotAssignedf(format string, args ...interface{}) error
```
NotAssignedf returns an error which satisfies IsNotAssigned().


## func NotFoundf
``` go
func NotFoundf(format string, args ...interface{}) error
```
NotFoundf returns an error which satisfies IsNotFound().


## func NotImplementedf
``` go
func NotImplementedf(format string, args ...interface{}) error
```
NotImplementedf returns an error which satisfies IsNotImplemented().


## func NotProvisionedf
``` go
func NotProvisionedf(format string, args ...interface{}) error
```
NotProvisionedf returns an error which satisfies IsNotProvisioned().


## func NotSupportedf
``` go
func NotSupportedf(format string, args ...interface{}) error
```
NotSupportedf returns an error which satisfies IsNotSupported().


## func NotValidf
``` go
func NotValidf(format string, args ...interface{}) error
```
NotValidf returns an error which satisfies IsNotValid().


## func Trace
``` go
func Trace(other error) error
```
Trace adds the location of the Trace call to the stack.  The Cause of the
resulting error is the same as the error parameter.  If the other error is
nil, the result will be nil.

For example:


	if err := SomeFunc(); err != nil {
	    return errors.Trace(err)
	}


## func Unauthorizedf
``` go
func Unauthorizedf(format string, args ...interface{}) error
```
Unauthorizedf returns an error which satisfies IsUnauthorized().


## func UserNotFoundf
``` go
func UserNotFoundf(format string, args ...interface{}) error
```
UserNotFoundf returns an error which satisfies IsUserNotFound().


## func Wrap
``` go
func Wrap(other, newDescriptive error) error
```
Wrap changes the Cause of the error. The location of the Wrap call is also
stored in the error stack.

For example:


	if err := SomeFunc(); err != nil {
	    newErr := &packageError{"more context", private_value}
	    return errors.Wrap(err, newErr)
	}


## func Wrapf
``` go
func Wrapf(other, newDescriptive error, format string, args ...interface{}) error
```
Wrapf changes the Cause of the error, and adds an annotation. The location
of the Wrap call is also stored in the error stack.

For example:


	if err := SomeFunc(); err != nil {
	    return errors.Wrapf(err, simpleErrorType, "invalid value %q", value)
	}



## type Err
``` go
type Err struct {
    // contains filtered or unexported fields
}
```
Err holds a description of an error along with information about
where the error was created.

It may be embedded in custom error types to add extra information that
this errors package can understand.









### func NewErr
``` go
func NewErr(format string, args ...interface{}) Err
```
NewErr is used to return an Err for the purpose of embedding in other
structures.  The location is not specified, and needs to be set with a call
to SetLocation.

For example:


	type FooError struct {
	    errors.Err
	    code int
	}
	
	func NewFooError(code int) error {
	    err := &FooError{errors.NewErr("foo"), code}
	    err.SetLocation(1)
	    return err
	}




### func (\*Err) Cause
``` go
func (e *Err) Cause() error
```
The Cause of an error is the most recent error in the error stack that
meets one of these criteria: the original error that was raised; the new
error that was passed into the Wrap function; the most recently masked
error; or nil if the error itself is considered the Cause.  Normally this
method is not invoked directly, but instead through the Cause stand alone
function.



### func (\*Err) Error
``` go
func (e *Err) Error() string
```
Error implements error.Error.



### func (\*Err) Location
``` go
func (e *Err) Location() (filename string, line int)
```
Location is the file and line of where the error was most recently
created or annotated.



### func (\*Err) Message
``` go
func (e *Err) Message() string
```
Message returns the message stored with the most recent location. This is
the empty string if the most recent call was Trace, or the message stored
with Annotate or Mask.



### func (\*Err) SetLocation
``` go
func (e *Err) SetLocation(callDepth int)
```
SetLocation records the source location of the error at callDepth stack
frames above the call.



### func (\*Err) StackTrace
``` go
func (e *Err) StackTrace() []string
```
StackTrace returns one string for each location recorded in the stack of
errors. The first value is the originating error, with a line for each
other annotation or tracing of the error.



### func (\*Err) Underlying
``` go
func (e *Err) Underlying() error
```
Underlying returns the previous error in the error stack, if any. A client
should not ever really call this method.  It is used to build the error
stack and should not be introspected by client calls.  Or more
specifically, clients should not depend on anything but the `Cause` of an
error.









- - -
Generated by [godoc2md](http://godoc.org/github.com/davecheney/godoc2md)