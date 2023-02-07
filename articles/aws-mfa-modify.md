---

title: "MFA仕様変更で運用やポリシー設定が変わった"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Security"]
published: true

---

# 10秒で概要

- AWSのMFAの仕様変更により、MFAを利用した運用手順やポリシーの設定に変更が必要です。
  - [AWS Identity and Access Management で複数の多要素認証 (MFA) デバイスのサポートを開始](https://aws.amazon.com/jp/about-aws/whats-new/2022/11/aws-identity-access-management-multi-factor-authentication-devices/)
- ポリシーの設定および、stsトークンを取得する際のコマンドに注意が必要です。

# 仕様変更の概要

- 通常IAMユーザがAWSリソースを触る際は、コンソールへのログイン時／CLIからの操作時にMFA（二要素認証）を取り入れることで、よりセキュアなID管理ができます。
- このMFA用のデバイスについて、1つのIAMユーザに複数のMFAデバイスを割り当てられるようになりました。

# ポリシーの変更

以下[公式で提供](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_policies_examples_iam_mfa-selfmanage.html)される、MFA用のデバイス割り当てた際のポリシー設定です。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowListActions",
            "Effect": "Allow",
            "Action": [
                "iam:ListUsers",
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowIndividualUserToManageTheirOwnMFA",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:ListMFADevices",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/${aws:username}",
                "arn:aws:iam::*:user/${aws:username}"
            ]
        },
        {
            "Sid": "AllowIndividualUserToDeactivateOnlyTheirOwnMFAOnlyWhenUsingMFA",
            "Effect": "Allow",
            "Action": [
                "iam:DeactivateMFADevice"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/${aws:username}",
                "arn:aws:iam::*:user/${aws:username}"
            ],
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        },
        {
            "Sid": "BlockMostAccessUnlessSignedInWithMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:ListMFADevices",
                "iam:ListUsers",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

上から順に、

- （不要）`AllowListActions`
  - IAMユーザの一覧、仮想MFAデバイスのリスト表示を許可します。
  - これは、コンソールログイン後右上の自己メニュー内、`セキュリティ認証情報`から遷移できるため不要です。むしろ、他人のIAM一覧が見えてしまうので無くても良いです。
- （必要）`AllowIndividualUserToManageTheirOwnMFA`
  - 自分のIAMユーザに限り、MFAデバイスの追加／削除／有効化／再同期を許可します。
- （任意）`AllowIndividualUserToDeactivateOnlyTheirOwnMFAOnlyWhenUsingMFA`
  - MFAデバイスが有効な場合に限り、MFAデバイスの無効化を許可します。
- （必要）`BlockMostAccessUnlessSignedInWithMFA`
  - MFAデバイスが有効でない場合に、MFAデバイスの追加に関する操作以外は拒否します。

以上より、必須な設定のみ抜き出すと以下の設定となります。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowIndividualUserToManageTheirOwnMFA",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListMFADevices",
                "iam:ChangePassword"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/${aws:username}",
                "arn:aws:iam::*:user/${aws:username}"
            ]
        },
        {
            "Sid": "BlockMostAccessUnlessSignedInWithMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListMFADevices",
                "iam:ChangePassword"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

上記の設定のままMFA設定者が操作を行うと、以下の手順でエラーが出るようになりました。

- MFAデバイスを１台しか割り当てられなかった時とは異なり、デバイス名の入力を求められる。
  ![mfa1](/images/aws-mfa-modify/MFA1.png)
- 次へを押下すると、これまでは出なかったエラーが表示される。
  ![mfa2](/images/aws-mfa-modify/MFA2.png)

今回の仕様変更により問題になったのは、`AllowIndividualUserToManageTheirOwnMFA.Resource`内の`"arn:aws:iam::*:mfa/${aws:username}"`部分です。
これまではMFAデバイスを１台のみ割り当てることが可能だったため、自分のIAMユーザ名称を固定値でそのままMFAデバイスの識別子名称として利用していました。MFAデバイスに任意の識別子名称を付けられるようになったことで、識別子の名称として自分のIAMユーザの名称を利用しなかった場合に、権限エラーが出てしまいます。

よって、Jsonは以下のように変更されます。
こうすることで、MFAデバイスの識別子がなんであれ、自分のMFAの設定を進めることが可能になりました。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowIndividualUserToManageTheirOwnMFA",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListMFADevices",
                "iam:ChangePassword"
            ],
            "Resource": [
                "arn:aws:iam::*:mfa/*", // ここを修正
                "arn:aws:iam::*:user/${aws:username}"
            ]
        },
        {
            "Sid": "BlockMostAccessUnlessSignedInWithMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice",
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ResyncMFADevice",
                "iam:ListMFADevices",
                "iam:ChangePassword"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

# コンソールからのstsトークン取得時の注意

MFAを利用している場合、コンソールからAWSリソースを触る場合にはSTSコマンドによりsession tokenを取得する必要が有りました。
`$ aws sts get-session-token --serial-number arn:aws:iam::999999999999:mfa/own-user --token-code 000000 --profile hogehoge`

この中で、`mfa/own-user`の部分が、これまでIAMユーザの名称固定で設定されるように設定していましたが、今後は任意に設定したMFAデバイスの識別子を利用する必要が有ります。つまり、前章にて識別子を`googleAuthentication`のように設定していた場合、コマンドは以下のようになります。

`$ aws sts get-session-token --serial-number arn:aws:iam::999999999999:mfa/googleAuthentication --token-code 000000 --profile hogehoge`

# 終わりに

- AWSのMFAの仕様変更により、ポリシーの修正が必要です。
- ポリシーの修正に伴い、session-tokenの取得時にコマンド上注意が必要です。
- そもそもMFAデバイスを割り当てないような運用はやめましょう。キーが漏洩した際の被害が甚大です。
