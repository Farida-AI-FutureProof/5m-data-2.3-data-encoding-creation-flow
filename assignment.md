# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```python
# Thrift schema (student.thrift)

%%writefile ../schema/student.thrift

struct Student {
  1: required string name,
  2: required i64 age,
  3: required list<string> courses
}

service School {
  Student enrollCourse(1: required Student student, 2: required string courseName)
}

# Thrift server (student_server.py)

%%writefile ../student_server.py
import thriftpy2
from thriftpy2.rpc import make_server

student_thrift = thriftpy2.load("./schema/student.thrift", module_name="student_thrift")

class School(object):
    def enrollCourse(self, student, courseName):
        student.courses.append(courseName)
        return student

if __name__ == "__main__":
    print("Starting Thrift server on port 6000...")
    server = make_server(student_thrift.School, School(), host="127.0.0.1", port=6000)
    server.serve()

# Thrift client (student_client.py)

%%writefile ../student_client.py
import thriftpy2
from thriftpy2.rpc import make_client

student_thrift = thriftpy2.load("./schema/student.thrift", module_name="student_thrift")

client = make_client(student_thrift.School, "127.0.0.1", 6000)

student = student_thrift.Student(
    name="Farida",
    age=48,
    courses=["Data Science"]
)

updated = client.enrollCourse(student, "AI Engineering")
print(updated)

```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```python
# Protobuf schema (book.proto)

%%writefile ../schema/book.proto
syntax = "proto3";

message Book {
  string title = 1;
  string author = 2;
  int32 page_count = 3;
}

service Library {
  rpc checkoutBook(Book) returns (Book) {}
}
# Command to generate protobuf Python files:
# python -m grpc_tools.protoc -I./schema --python_out=. --grpc_python_out=. ./schema/book.proto

# Protobuf server (book_server.py)

%%writefile ../book_server.py
import grpc
from concurrent import futures
import book_pb2
import book_pb2_grpc

class Library(book_pb2_grpc.LibraryServicer):
    def checkoutBook(self, request, context):
        return request   # return the same Book

if __name__ == "__main__":
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=2))
    book_pb2_grpc.add_LibraryServicer_to_server(Library(), server)
    server.add_insecure_port("[::]:50051")
    server.start()
    print("gRPC server running on port 50051...")
    server.wait_for_termination()

# Protobuf client (book_client.py)

%%writefile ../book_client.py
import grpc
import book_pb2
import book_pb2_grpc

with grpc.insecure_channel("localhost:50051") as channel:
    stub = book_pb2_grpc.LibraryStub(channel)
    
    book = book_pb2.Book(
        title="The Great Gatsby",
        author="F. Scott Fitzgerald",
        page_count=180
    )
    
    result = stub.checkoutBook(book)
    
    print("Title:", result.title)
    print("Author:", result.author)
    print("Page Count:", result.page_count)

```

## Submission

- Used GitHub Copilot to generate initial drafts of book.proto, student_client.py, student_server.py, and student.thrift.
- Used ChatGPT to validate and correct the code.
- Executed and tested the code successfully on Google Colab.

