# Introduction
We will learn how to run a basic _Node.js_ HTTP server and setup our project structure using the _Express_ framework.
Secondly, we will understand how to create the Model of our project (from the MVC pattern) and learn how the _Sequelize_ package will help us creating the relational database schema and perform operations on the Maria Database.
## Prerequisites
* Keep in mind we are developing the backend software needed for DeliverUS project. Please, read project requirements found at: https://github.com/IISSI2-IS/DeliverUS-Backend/blob/main/README.md
* Software requirements for the developing environment con be found at: https://github.eii.us.es/IISSI2-IS/IISSI2-IS-Backend/wiki
  * The template project includes EsLint configuration so it should auto-fix formatting problems as soon as a file is saved.






controllers/ProductController.js

  }
    try {
      newProduct = await newProduct.save()
      updateRestaurantInexpensiveness(newProduct.restaurantId)
      res.json(newProduct)
    } catch (err) {
      if (err.name.includes('ValidationError')) {
@@ -43,6 +44,31 @@ exports.create = async function (req, res) {
  }
}

const updateRestaurantInexpensiveness = async function (restaurantId) {
  const queryResultOtherRestaurantsAvgPrice = await Product.findOne({
    where: {
      restaurantId: { [Sequelize.Op.ne]: restaurantId }
    },
    attributes: [
      [Sequelize.fn('AVG', Sequelize.col('price')), 'avgPrice']
    ]
  })
  const queryResultCurrentRestaurantAvgPrice = await Product.findOne({
    where: {
      restaurantId: restaurantId
    },
    attributes: [
      [Sequelize.fn('AVG', Sequelize.col('price')), 'avgPrice']
    ]
  })
  if (queryResultCurrentRestaurantAvgPrice !== null && queryResultOtherRestaurantsAvgPrice !== null) {
    const avgPriceOtherRestaurants = queryResultOtherRestaurantsAvgPrice.dataValues.avgPrice
    const avgPriceCurrentRestaurant = queryResultCurrentRestaurantAvgPrice.dataValues.avgPrice
    const isInexpensive = avgPriceCurrentRestaurant < avgPriceOtherRestaurants
    Restaurant.update({ isInexpensive: isInexpensive }, { where: { id: restaurantId } })
  }
}

exports.update = async function (req, res) {
  const err = validationResult(req)
  if (err.errors.length > 0) {

