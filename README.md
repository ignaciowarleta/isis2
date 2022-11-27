# Introduction
We will learn how to run a basic _Node.js_ HTTP server and setup our project structure using the _Express_ framework.
Secondly, we will understand how to create the Model of our project (from the MVC pattern) and learn how the _Sequelize_ package will help us creating the relational database schema and perform operations on the Maria Database.
## Prerequisites
* Keep in mind we are developing the backend software needed for DeliverUS project. Please, read project requirements found at: https://github.com/IISSI2-IS/DeliverUS-Backend/blob/main/README.md
* Software requirements for the developing environment con be found at: https://github.eii.us.es/IISSI2-IS/IISSI2-IS-Backend/wiki
  * The template project includes EsLint configuration so it should auto-fix formatting problems as soon as a file is saved.






BACKEND

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






controllers/RestaurantController.js


try {
    const restaurants = await Restaurant.findAll(
      {
        attributes: ['id', 'name', 'description', 'address', 'postalCode', 'url', 'shippingCosts', 'averageServiceMinutes', 'email', 'phone', 'logo', 'heroImage', 'status', 'restaurantCategoryId'],
        attributes: ['id', 'name', 'description', 'address', 'postalCode', 'url', 'shippingCosts', 'averageServiceMinutes', 'email', 'phone', 'logo', 'heroImage', 'status', 'restaurantCategoryId', 'isInexpensive'],
        include:
      {
        model: RestaurantCategory,
@@ -29,7 +29,7 @@ exports.indexOwner = async function (req, res) {
  try {
    const restaurants = await Restaurant.findAll(
      {
        attributes: ['id', 'name', 'description', 'address', 'postalCode', 'url', 'shippingCosts', 'averageServiceMinutes', 'email', 'phone', 'logo', 'heroImage', 'status', 'restaurantCategoryId'],
        attributes: ['id', 'name', 'description', 'address', 'postalCode', 'url', 'shippingCosts', 'averageServiceMinutes', 'email', 'phone', 'logo', 'heroImage', 'status', 'restaurantCategoryId', 'isInexpensive'],
        where: { userId: req.user.id }
      })
    res.json(restaurants)
    
    
    
    
     
migrations/20210629195916-create-restaurant.js
@@ -57,6 +57,10 @@ module.exports = {
        ],
        defaultValue: 'offline'
      },
      isInexpensive: {
        type: Sequelize.BOOLEAN,
        defaultValue: false
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE,
  4  
models/restaurant.js
@@ -50,6 +50,10 @@ module.exports = (sequelize, DataTypes) => {
        'temporarily closed'
      ]
    },
    isInexpensive: {
      type: DataTypes.BOOLEAN,
      defaultValue: false
    },
    restaurantCategoryId: {
      allowNull: false,
      type: DataTypes.INTEGER
      
      
      
      
      
      
      
      
      
      
      
      FRONTEND
      
      src/screens/restaurants/RestaurantsScreen.js
      
      
      
          navigation.navigate('RestaurantDetailScreen', { id: item.id })
        }}
      >

        <TextRegular numberOfLines={2}>{item.description}</TextRegular>
        {item.averageServiceMinutes !== null &&
          <TextSemiBold>Avg. service time: <TextSemiBold textStyle={{ color: brandPrimary }}>{item.averageServiceMinutes} min.</TextSemiBold></TextSemiBold>
        }
        <TextSemiBold>Shipping: <TextSemiBold textStyle={{ color: brandPrimary }}>{item.shippingCosts.toFixed(2)}€</TextSemiBold></TextSemiBold>
         {/* SOLUTION */}
        <View style={{ flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center' }} >
            <TextSemiBold>Shipping: <TextSemiBold textStyle={{ color: brandPrimary }}>{item.shippingCosts.toFixed(2)}€</TextSemiBold></TextSemiBold>
            {item.isInexpensive &&
                <TextRegular textStyle={[styles.badge, { color: brandSuccess, borderColor: brandSuccess }] }>€</TextRegular>
            }
            {!item.isInexpensive &&
            <TextRegular textStyle={[styles.badge, { color: brandPrimary, borderColor: brandPrimary }] }>€€</TextRegular>
            }
        </View>
         {/* END SOLUTION */}
      </ImageCard>
    )
  }
@@ -120,5 +131,14 @@ const styles = StyleSheet.create({
  emptyList: {
    textAlign: 'center',
    padding: 50
  },
  // Solucion
  badge: {
    textAlign: 'center',
    borderWidth: 2,
    width: 45,
    paddingVertical: 2,
    paddingHorizontal: 10,
    borderRadius: 10
  }
})
