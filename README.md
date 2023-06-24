- 👋 Hi, I’m @Tomroger2525
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Tomroger2525/Tomroger2525 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
วงจรแพ็คเกจ

นำเข้า (
	"github.com/binance/zkmerkle-proof-of-solvency/src/utils"
	"github.com/consensys/gnark/std/hash/โพไซดอน"
)

พิมพ์โครงสร้าง BatchCreateUserCircuit {
	ตัวแปร BatchCommitment `gnark:", สาธารณะ"`
	ตัวแปร BeforeAccountTreeRoot
	ตัวแปร AfterAccountTreeRoot
	ก่อน CEXAssetsCommitment Variable
	AfterCEXAssetsCommitment Variable
	ก่อน CexAssets [] CexAssetInfo
	CreateUserOps [] CreateUserOperation
}

func NewVerifyBatchCreateUserCircuit (ความมุ่งมั่น [] ไบต์) * BatchCreateUserCircuit {
	var v BatchCreateUserCircuit
	v.BatchCommitment = ความมุ่งมั่น
	ย้อนกลับ &v
}

func NewBatchCreateUserCircuit (assetCounts uint32, batchCounts uint32) * BatchCreateUserCircuit {
	วงจร var BatchCreateUserCircuit
	วงจร BatchCommitment = 0
	วงจรก่อนบัญชีต้นไม้รูท = 0
	วงจร AfterAccountTreeRoot = 0
	วงจรก่อน CEXAssetsCommitment = 0
	วงจร AfterCEXAssetsCommitment = 0
	วงจรก่อน CexAssets = make ([] CexAssetInfo, assetCounts)
	สำหรับฉัน := uint32(0); ฉัน <assetCounts; ฉัน ++ {
		วงจรก่อน CexAssets [i] = CexAssetInfo{
			TotalEquity: 0,
			หนี้ทั้งหมด: 0,
			ราคาพื้นฐาน: 0,
		}
	}
	วงจร CreateUserOps = ทำ ([] CreateUserOperation, batchCounts)
	สำหรับฉัน := uint32(0); ฉัน < batchCounts; ฉัน ++ {
		วงจร CreateUserOps[i] = CreateUserOperation{
			ก่อนบัญชีต้นไม้ราก: 0,
			AfterAccountTreeRoot: 0,
			สินทรัพย์: make([]UserAssetInfo,assetCounts),
			ดัชนีบัญชี: 0,
			AccountProof: [utils.AccountTreeDepth]ตัวแปร{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 , 0, 0, 0, 0, 0, 0, 0},
		}
		สำหรับ j := uint32(0); j <assetCounts; j ++ {
			วงจร CreateUserOps[i].Assets[j].Debt = 0
			circuit.CreateUserOps[i].Assets[j].Equity = 0
		}
	}
	ส่งคืน & วงจร
}

ข้อผิดพลาด func (b BatchCreateUserCircuit) กำหนด (api API) {
	// ตรวจสอบว่า BatchCommitment คำนวณถูกต้องหรือไม่
	actualBatchCommitment := poseidon.Poseidon(api, b.BeforeAccountTreeRoot, b.AfterAccountTreeRoot, b.BeforeCEXAssetsCommitment, b.AfterCEXAssetsCommitment)
	api.AssertIsEqual (b.BatchCommitment, จริง BatchCommitment)
	cexAssets := make([]Variable, len(b.BeforeCexAssets))
	afterCexAssets := make([]CexAssetInfo, len(b.BeforeCexAssets))

	// ตรวจสอบว่า beforeCexAssetsCommitment คำนวณถูกต้องหรือไม่
	สำหรับฉัน := 0; ฉัน < len (b.BeforeCexAssets); ฉัน ++ {
		CheckValueInRange(api, b.BeforeCexAssets[i].TotalEquity)
		CheckValueInRange(api, b.BeforeCexAssets[i].TotalDebt)
		CheckValueInRange(api, b.Before CexAssets[i].BasePrice)
		cexAssets[i] = api.Add(api.Mul(b.BeforeCexAssets[i].TotalEquity, utils.Uint64MaxValueFrSquare),
			api.Mul(b.BeforeCexAssets[i].TotalDebt, utils.Uint64MaxValueFr), b.BeforeCexAssets[i].BasePrice)
		afterCexAssets[i] = b.BeforeCexAssets[i]
	}
	actualCexAssetsCommitment := poseidon.Poseidon(api, cexAssets...)
	api.AssertIsEqual(b.Before CEXAssetsCommitment จริง CexAssetsCommitment)

	api.AssertIsEqual(b.BeforeAccountTreeRoot, b.CreateUserOps[0].BeforeAccountTreeRoot)
	api.AssertIsEqual(b.AfterAccountTreeRoot, b.CreateUserOps[len(b.CreateUserOps)-1].AfterAccountTreeRoot)

	tempAfterCexAssets := make([]Variable, len(b.BeforeCexAssets))
	สำหรับฉัน := 0; ฉัน < len (b.CreateUserOps); ฉัน ++ {
		accountIndexHelper := AccountIdToMerkleHelper(api, b.CreateUserOps[i].AccountIndex)
		VerifyMerkleProof(api, b.CreateUserOps[i].BeforeAccountTreeRoot, EmptyAccountLeafNodeHash, b.CreateUserOps[i].AccountProof[:], accountIndexHelper)
		var ตัวแปร totalUserEquity = 0
		var ตัวแปร totalUserDebt = 0
		userAssets := b.CreateUserOps[i].Assets
		สำหรับ j := 0; j < เลนส์ (userAssets); j ++ {
			CheckValueInRange(api, userAssets[j].Debt)
			CheckValueInRange(api, userAssets[j].Equity)
			totalUserEquity = api.Add(totalUserEquity, api.Mul(userAssets[j].Equity, b.BeforeCexAssets[j].BasePrice))
			totalUserDebt = api.Add(totalUserDebt, api.Mul(userAssets[j].Debt, b.BeforeCexAssets[j].BasePrice))

			afterCexAssets[j].TotalEquity = api.Add(afterCexAssets[j].TotalEquity, userAssets[j].Equity)
			afterCexAssets[j].TotalDebt = api.Add(afterCexAssets[j].TotalDebt, userAssets[j].Debt)
		}
		// ตรวจสอบให้แน่ใจว่า Equity ทั้งหมดของผู้ใช้มากกว่าหรือเท่ากับหนี้สินทั้งหมดของผู้ใช้
		api.AssertIsLessOrEqual (totalUserDebt, totalUserEquity)

		userAssetsCommitment := ComputeUserAssetsCommitment(api, userAssets)
		accountHash := poseidon.Poseidon(api, b.CreateUserOps[i].AccountIdHash, totalUserEquity, totalUserDebt, userAssetsCommitment)
		actualAccountTreeRoot := UpdateMerkleProof(api, accountHash, b.CreateUserOps[i].AccountProof[:], accountIndexHelper)
		api.AssertIsEqual (actualAccountTreeRoot, b.CreateUserOps[i].AfterAccountTreeRoot)

	}

	สำหรับ j := 0; j < len (tempAfterCexAssets); j ++ {
		CheckValueInRange(api, afterCexAssets[j].TotalEquity)
		CheckValueInRange(api, afterCexAssets[j].TotalDebt)
		tempAfterCexAssets[j] = api.Add(api.Mul(afterCexAssets[j].TotalEquity, utils.Uint64MaxValueFrSquare),
			api.Mul(afterCexAssets[j].TotalDebt, utils.Uint64MaxValueFr), afterCexAssets[j].BasePrice)
	}

	// ตรวจสอบ AfterCEXAssetsCommitment ว่าคำนวณถูกต้อง
	actualAfterCEXAssetsCommitment := poseidon.Poseidon(api, tempAfterCexAssets...)
	api.AssertIsEqual (ตามจริง AfterCEXAssetsCommitment, b.AfterCEXAssetsCommitment)

	สำหรับฉัน := 0; ฉัน < len (b.CreateUserOps) -1; ฉัน ++ {
		api.AssertIsEqual(b.CreateUserOps[i].AfterAccountTreeRoot, b.CreateUserOps[i+1].BeforeAccountTreeRoot)
	}

	กลับเป็นศูนย์
}

func SetBatchCreateUserCircuitWitness (batchWitness * utils.BatchCreateUserWitness) (พยาน * BatchCreateUserCircuit ผิดพลาด) {
	พยาน = &BatchCreateUserCircuit{
		BatchCommitment: batchWitness.BatchCommitment,
		BeforeAccountTreeRoot: batchWitness.BeforeAccountTreeRoot,
		AfterAccountTreeRoot: batchWitness.AfterAccountTreeRoot,
		ก่อน CEXAssetsCommitment: batchWitness ก่อน CEXAssetsCommitment
		AfterCEXAssetsCommitment: batchWitness.AfterCEXAssetsCommitment,
		BeforeCexAssets: make([]CexAssetInfo, len(batchWitness.BeforeCexAssets)),
		CreateUserOps: make([]CreateUserOperation, len(batchWitness.CreateUserOps)),
	}

	สำหรับฉัน := 0; ฉัน < len (พยานก่อน CexAssets); ฉัน ++ {
		พยานก่อน CexAssets[i].TotalEquity = batchWitness.BeforeCexAssets[i].TotalEquity
		พยานก่อน CexAssets[i].TotalDebt = batchWitness.BeforeCexAssets[i].TotalDebt
		พยานก่อน CexAssets[i].BasePrice = batchWitness.Before CexAssets[i].BasePrice
	}

	สำหรับฉัน := 0; ฉัน < len (พยาน.CreateUserOps); ฉัน ++ {
		Witness.CreateUserOps[i].BeforeAccountTreeRoot = batchWitness.CreateUserOps[i].BeforeAccountTreeRoot
		พยาน.CreateUserOps[i].AfterAccountTreeRoot = batchWitness.CreateUserOps[i].AfterAccountTreeRoot
		พยาน.CreateUserOps[i].Assets = make([]UserAssetInfo, len(batchWitness.CreateUserOps[i].Assets))
		สำหรับ j := 0; j < len (batchWitness.CreateUserOps[i].Assets); j ++ {
			var userAsset UserAssetInfo
			userAsset.Equity = batchWitness.CreateUserOps[i].Assets[j].Equity
			userAsset.Debt = batchWitness.CreateUserOps[i].Assets[j].Debt
			พยาน CreateUserOps[i].Assets[j] = userAsset
		}
		พยาน CreateUserOps[i].AccountIdHash = batchWitness.CreateUserOps[i].AccountIdHash
		พยาน.CreateUserOps[i].AccountIndex = batchWitness.CreateUserOps[i].AccountIndex
		สำหรับ j := 0; j < len (พยาน.CreateUserOps[i].AccountProof); j ++ {
			พยาน.CreateUserOps[i].AccountProof[j] = batchWitness.CreateUserOps[i].AccountProof[j]
		}
	}
	พยานกลับศูนย์
