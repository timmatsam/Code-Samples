<h3>This is a code sample from my ecommerce website project.</h3>

```js
router.get('/', async (req, res) => {
  try {
    const cart = await Order.findOne({
      where: {isOrder: false, userId: req.user.id},
      include: Product
    })
    if (cart) {
      res.json(cart.Products)
    }
  } catch (error) {
    res.sendStatus(error)
  }
})

router.get('/orders', async (req, res, next) => {
  try {
    const orders = await Order.findAll({
      where: {isOrder: true, userId: req.user.id},
      include: Product
    })
    res.json(orders)
  } catch (error) {
    res.sendStatus(error)
  }
})

router.post('/addProduct/:productId', async (req, res) => {
  try {
    const productId = req.params.productId
    const [order] = await Order.findOrCreate({
      where: {isOrder: false, userId: req.user.id}
    })
    const product = await Product.findByPk(productId)
    const price = product.price * 100
    const isProductInCart = await Order_Product.findOne({
      where: {ProductId: productId, OrderId: order.id}
    })
    if (isProductInCart) {
      isProductInCart.quantity += 1
      isProductInCart.price += price
      await isProductInCart.save()
    } else {
      await order.addProduct(product, {through: {price: price}})
    }

    const cart = await Order.findOne({
      where: {isOrder: false},
      include: Product
    })
    res.json(product)
  } catch (error) {
    res.sendStatus(error)
  }
})

router.delete('/removeProduct/:productId', async (req, res) => {
  try {
    const productId = req.params.productId
    const order = await Order.findOne({
      where: {isOrder: false, userId: req.user.id}
    })

    const isproductInCart = await Order_Product.findOne({
      where: {ProductId: productId, OrderId: order.id}
    })
    await isproductInCart.destroy()

    res.json(isproductInCart)
  } catch (error) {
    res.sendStatus(error)
  }
})

router.put('/updateQuantity/:productId', async (req, res) => {
  try {
    const productId = req.params.productId
    const increment = req.body.increment
    const product = await Product.findByPk(productId)
    const price = product.price * 100
    const order = await Order.findOne({
      where: {isOrder: false, userId: req.user.id}
    })
    const productInCart = await Order_Product.findOne({
      where: {ProductId: productId, OrderId: order.id}
    })
    if (increment) {
      productInCart.quantity += 1
      productInCart.price += price

      await productInCart.save()
    } else {
      productInCart.quantity -= 1
      productInCart.price -= price
      await productInCart.save()
    }
    if (productInCart.quantity === 0) {
      await productInCart.destroy()
      res.json(productInCart)
    } else res.json(productInCart.quantity)
  } catch (error) {
    res.sendStatus(error)
  }
})

router.get('/checkout', async (req, res) => {
  try {
    const orderToBeCheckedOut = await Order.findOne({
      where: {isOrder: false, userId: req.user.id},
      include: Product
    })

    if (orderToBeCheckedOut.Products.length > 0) {
      orderToBeCheckedOut.isOrder = true
      await orderToBeCheckedOut.save()
      await Order.create({userId: req.user.id})
      res.json(orderToBeCheckedOut)
    } else res.sendStatus(400)
  } catch (error) {
    res.sendStatus(error)
  }
})
