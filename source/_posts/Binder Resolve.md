title: Binder Resolve
date: 2019-1-17
tags: [Android]
categories: Android开发艺术探索
description: 　　
---
```java
public interface IBookManager extends IInterface {
  static final String DESCRIPTOR = "com.hdu.test.IBookManager";

  static final int TRANSACTION_getBOOKLIST = IBinder.FIRST_CALL_TRANSACTION + 0;// Flag the method invoked
  static final int TRANSACTION_addBook = IBinder.FIRST_CALL_TRANSACTION + 1;

  public List<Book> getBookList() throws RemoteException;// method 

  public void addBook(Book book) throws RemoteException;

  public class BookManagetImpl extends Binder implements IBookManager {//The concrete  implementation  of Binder

    public BookManagetImpl() {
        this.attachInterface(this, DESCRIPTOR);// Register this IInterface,so that queryLocalInterface would use DESCRIPTOR to get IInterface.
    }

    public static IBookManager asInterface(IBinder binder) {// Client invoke this method to obtain IBookManager inteface
      if (binder == null) {
        return null;
      }
      IInterface iin = binder.queryLocalInterface(DESCRIPTOR);// Obtain IInterface by DESCRIPTOR.
      if ((iin != null) && (iin instanceof IBookManager)) {
        return (IBookManager) iin;
      }
      return new BookManagetImpl.Proxy(binder);
    }

    @Override protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {//server would progress RPC
      switch (code) {
        case INTERFACE_TRANSACTION:
          reply.writeString(DESCRIPTOR);
          return true;
        case TRANSACTION_getBOOKLIST:
          data.enforceInterface(DESCRIPTOR);
          List<Book> books = this.getBookList();
          reply.writeNoException();
          reply.writeTypedList(books);//write reslut into reply
          return true;
        case TRANSACTION_addBook:
          data.enforceInterface(DESCRIPTOR);
          Book book = null;
          if (0 != data.readInt()) {
            book = Book.CREATOR.createFromParcel(data);
          }
          this.addBook(book);
          reply.writeNoException();
          return true;
      }
      return super.onTransact(code, data, reply, flags);
    }

    @Override public List<Book> getBookList() throws RemoteException {
      return null;
    }

    @Override public void addBook(Book book) throws RemoteException {

    }

    @Override public IBinder asBinder() {
      return this;
    }

    private static class Proxy implements IBookManager {

      private IBinder mRemote;

      public Proxy(IBinder remote) {
        mRemote = remote;
      }

      @Override public List<Book> getBookList() throws RemoteException {// Client would invoke these method  as follows
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        List<Book> books;
        try {
          data.writeInterfaceToken(DESCRIPTOR);
          mRemote.transact(TRANSACTION_getBOOKLIST, data, reply, 0);
          data.readException();
          books = reply.createTypedArrayList(Book.CREATOR);
        } finally {
          data.recycle();
          reply.recycle();
        }
        return books;
      }

      @Override public void addBook(Book book) throws RemoteException {  
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        try {
          data.writeInterfaceToken(DESCRIPTOR);
          if (book != null) {
            data.writeInt(1);
            book.writeToParcel(data, 0);
          } else {
            data.writeInt(0);
          }
          mRemote.transact(TRANSACTION_addBook, data, reply, 0);
          reply.readException();
        }finally {
          reply.recycle();
          data.recycle();
        }
      }

      @Override public IBinder asBinder() {
        return mRemote;
      }

      public String getInterfaceDescriptor() {
        return DESCRIPTOR;
      }
    }
  }
}

```
整个Binder运行机制如下图所示：
![Binder](/images/Binder.png)

1. 首先客户端通过IBookManager.asInteface()获得IBookManager中的Binder中的Proxy实例
2. 客户端调用Proxy的getBookList()，通过调用transcation()传入code、输入data、输出reply。发起了RPC请求。
3. 服务端通过onTranscation()处理RPC请求，根据传入的code值进行响应处理。若需要返回值，并将返回值写入reply.
4. 客户端若需要返回值则从reply中取出。