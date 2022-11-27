# laboratory-exam-backend-solutions
Solutions for the back-end part. Each turn and version of the solution will have the following format: morning/afternoon/english-version-X-version-description (e.g: morning-version-2-new-endpoints)
Enunciado: [Septiembre 2022-económicos.pdf](https://github.com/IISSI2-IS-profs/laboratory-exam-backend-solutions/files/9596841/Septiembre.2022-economicos.pdf)
Statement: [September 2022-inExpensiveness.pdf](https://github.com/IISSI2-IS-profs/laboratory-exam-backend-solutions/files/9596844/September.2022-inExpensiveness.pdf)

Please, mark with a comment // SOLUTION every part of the code added to solve the exam so it can be easily identified by the rest of profs.
Solutions for the back-end part










controllers/ProductController.js
@@ -32,6 +32,7 @@ exports.create = async function (req, res) {
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
@@ -10,7 +10,7 @@ exports.index = async function (req, res) {
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
      
      
      
      
      
      
      src/screens/restaurants/RestaurantsScreen.js
      
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
  
  
