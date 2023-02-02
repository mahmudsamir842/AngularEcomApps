# ECOMMERCE ANGULAR APP

## Description

This repository is a Software of Application with Angular.

## Installation

Using Angular 10 preferably.

## REST API (example)

https://github.com/DanielArturoAlejoAlvarez/ecommerce-rest-api-node

## Usage

```html
$ git clone https://github.com/DanielArturoAlejoAlvarez/ecommerce-angular-app.git
[NAME APP]

$ yarn install

$ ng serve

```

Follow the following steps and you're good to go! Important:

![alt text](https://www.belatrixsf.com/blog/wp-content/uploads/2017/11/image3.gif)

## Coding

### Config 

```ts
...
export class AuthComponent implements OnInit {
  formAuth: FormGroup;

  constructor(
    private _toastr: ToastrService,
    private _fb: FormBuilder,
    private _auth: AuthService,
    private _router: Router
  ) {
    this.formAuth = this._fb.group({
      username: ['', Validators.required],
      password: ['', Validators.required],
    });
  }

  ngOnInit(): void {}

  login() {
    this._auth.login(this.formAuth.value).subscribe((data) => {
      if (data.ok) {
        localStorage.setItem('token', data.token);
        this._toastr.success('You are now logged in!', 'SUCCESS')
        this._router.navigate(['user']);
      }else {
        //alert(data.msg)
        this._toastr.error(data.msg, 'ERROR')
      }
    },err=>{
      //alert(err.error.msg)
      this._toastr.error(err.error.msg, 'ERROR')
    });
  }
}
...
```

### Guards
```ts
...
export class AuthGuard implements CanActivate {

  constructor(private _as: AuthService, private _router: Router) {}

  canActivate(): boolean {
    if (this._as.loggedIn()) {
      return true
    }else {
      this._router.navigate(['login'])
      return false
    }
  }
  
}
...
```

### Services

```ts
...
export class OrderService {

  httpOptions = {
    headers: new HttpHeaders({
      'Authorization': localStorage.getItem('token')
    })
  }

  constructor(private _http: HttpClient) { }


  API_URI = 'http://127.0.0.1:3000/api/v1'

  getOrders(): Observable<any> {
    return this._http.get<Order[]>(`${this.API_URI}/orders`, this.httpOptions)
  }

  getOrder(id: number|string): Observable<any> {
    return this._http.get<Order>(`${this.API_URI}/orders/${id}`, this.httpOptions)
  }

  saveOrder(order: Order): Observable<any> {
    return this._http.post<Order>(`${this.API_URI}/orders`, order, this.httpOptions)
  }

  updateOrder(order: Order): Observable<any> {
    return this._http.put<Order>(`${this.API_URI}/orders/${order._id}`, order, this.httpOptions)
  }

  deleteOrder(order: Order): Observable<any> {
    return this._http.delete<Order>(`${this.API_URI}/orders/${order._id}`, this.httpOptions)
  }

}
...
```

### Components

```ts
...
export class OrderComponent implements OnInit {
  formOrder: FormGroup;

  orders = [];
  products = []
  
  clients = []

  edit: boolean = false;

  btnAction: string = 'CREATE';

  

  constructor(
    private _fb: FormBuilder,
    private _toastr: ToastrService,
    private _cs: ClientService,
    private _ps: ProductService,
    private _os: OrderService,
    private _router: Router
  ) {
    this.formOrder = this._fb.group({
      _id: [1],
      client: [{}],
      product: [{}],
      price: [0],
      qty: [1],
      total: [0],
      orderItems: [[]],
      serial: ['']
    });
  }

  ngOnInit(): void {
    
    this.orderList();
    this.clientList()
    this.productList()
    
  }

  putPrice() {
    let data = this.formOrder.value.product.split('-'); 
    this.formOrder.patchValue({
      price: data[1]
    })   
  }

  putSerialNumber() {
    const pos = this.clients.indexOf(this.formOrder.value.client)
    let serialNum= this.generateSerialNumber(pos)
    this.formOrder.value.serial = 
    this.formOrder.patchValue({
      serial: serialNum
    })  
  }

  deleteItem(item: number) {
    const items = this.formOrder.value.orderItems
    let obj = items.splice(items.indexOf(item), 1);
    this.formOrder.value.total -=  obj[0].subtotal    
  }


  addCart() {
    let data = this.formOrder.value.product.split('-');

    let exist = this.formOrder.value.orderItems.findIndex(e=>e.product_id == data[0])
    console.log(exist)

    if (exist != -1) {
      this.formOrder.value.orderItems[exist] = {
        product_id: data[0],
        product_name: data[2],
        qty: Number(this.formOrder.value.orderItems[exist].qty) + Number(this.formOrder.value.qty),
        price: this.formOrder.value.price,
        subtotal: this.formOrder.value.price * (Number(this.formOrder.value.orderItems[exist].qty) + Number(this.formOrder.value.qty))
      }
    } else {
      this.formOrder.value.orderItems.push({
        product_id: data[0],
        product_name: data[2],
        qty: this.formOrder.value.qty,
        price: this.formOrder.value.price,
        subtotal: this.formOrder.value.qty * this.formOrder.value.price
      });
    }

    
    this.formOrder.patchValue({
      total: this.formOrder.value.total + (this.formOrder.value.qty * this.formOrder.value.price)
    })  
  }

  orderList() {
    this._os.getOrders().subscribe(
      (data) => {
        console.log(data);
        this.orders = data.orders;
      },
      (err) => {
        console.log(err);
      }
    );
  }   

  addOrder() {
    this._os.saveOrder(this.formOrder.value)
      .subscribe(data=>{
        this.orderList()
        this.formOrder.reset({
          price: 0,
          qty: 1,
          total: 0
        })
        this._toastr.success('Order saved successfully!', 'SUCCESS')
      },err=>{
        console.log(err)
      })
  }

  clientList() {
    this._cs.getClients().subscribe(
      (data) => {
        console.log(data);
        this.clients = data.clients;
      },
      (err) => {
        console.log(err);
      }
    );
  }

  productList() {
    this._ps.getProducts().subscribe(
      (data) => {
        console.log(data);
        this.products = data.products;
      },
      (err) => {
        console.log(err);
      }
    );
  }


  generateSerialNumber(num: number) {
    if (num<=9) {
      return '000000000'+num
    }
    if (num<=99 && num >9) {
      return '00000000'+num
    }
    if (num<=999 && num >99) {
      return '0000000'+num
    }
    if (num<=9999 && num >999) {
      return '000000'+num
    }
    if (num<=99999 && num >9999) {
      return '00000'+num
    }
    if (num<=999999 && num >99999) {
      return '0000'+num
    }
    if (num<=9999999 && num >999999) {
      return '000'+num
    }
    if (num<=99999999 && num >999999) {
      return '00'+num
    }
    if (num<=999999999 && num >9999999) {
      return '0'+num
    }
  }
  
}
...
```

### Models 
```ts
...
export class Order {
  _id: number | string;
  client: Client;
  product: Product;
  price: number;
  qty: number;
  serial: number | string;
  total: number;
  orderItems: [];
}

...
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/DanielArturoAlejoAlvarez/ecommerce-angular-app. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
````
